---
author: ["Me"]
title: "How to set up dbt-core on MWAA"
description: "A quick guide to set up DBT on AWS Managed Apache Airflow (MWAA)"
summary: "Setting up DBT on MWAA is not as straightforward as it seems. This guide will discuss the different approaches and highlight the one that I found to be the most suitable for my use case."
tags: ["dbt", "airflow", "aws", "bash"]
categories: ["themes", "syntax"]
series: ["Themes Guide"]
ShowToc: true
TocOpen: true
editPost:
    URL: "https://github.com/ohm-s/306-dev-blog/edit/main/content/posts/dbt-mwaa-setup.md"
    Text: "Suggest Changes" 
    appendFilePath: false

---

### Why this guide?

DBT core is essential to run modern data pipelines, while there are many guides on how to set up DBT on Airflow, most of them focus on interoperating with dbt cloud, while others are not very suitable for MWAA. 

Furthermore, the [AWS documentation](https://docs.aws.amazon.com/mwaa/latest/userguide/samples-dbt.html) references an outdated version of DBT. And in general, the suggested approach of using a [`requirements.txt`](https://docs.aws.amazon.com/mwaa/latest/userguide/best-practices-dependencies.html) does not work great with the latest versions of DBT. One can easily run into Python dependency hell.

### Possible solutions

There are several approaches to tackle this problem; they are all valid and have their pros and cons. I will highlight a a couple of them and then focus on the one that I found to be the most suitable for my use case:

1- Using KubernetesPodOperator to run DBT in a containerized environment. 

This solution assumes that you are already maintaining an EKS Kubernetes cluster. It is the most flexible approach, it allows to update DBT and related Python versions & packages independently of your Airflow environment. However, this comes with the overhead of maintaining an image repository, mostly likely a different CI pipeline to build the image and delayed startup speed for the airflow DBT task. AWS provides an excellent [guide](https://docs.aws.amazon.com/mwaa/latest/userguide/mwaa-eks-example.html) on how to connect MWAA to EKS.

2- Using a startup script to install DBT on the MWAA worker nodes.

This https://docs.aws.amazon.com/mwaa/latest/userguide/using-startup-script.html is the easiest to implement and maintain. It is also the fastest way to spin up a new DBT task in Airflow. However, it is not the most flexible. You will have to update the startup script every time you want to update DBT. This is not a big deal if you are not updating DBT frequently.

### How to use the startup script approach

#### MWAA Internals

First off, let's understand a little bit more on how MWAA works under the hood. It maps your DAGS folder to (`/usr/local/airflow/dags`)[https://docs.aws.amazon.com/mwaa/latest/userguide/configuring-dag-folder.html#configuring-dag-folder-how] on the worker nodes. 

It also install the Python dependencies specified in `requirements.txt` to a different folder `/usr/local/airflow/.local/lib/python3.10/site-packages/`. This path might change depending on the MWAA version you are using, and hence the respective Python version.

To avoid mangling with Python dependencies, we are going to install the DBT libraries in a different folder. 

#### Steps to follow

- Create a new file called `startup.sh` in the root of your airflow DAGs folder. This file will be used to install DBT on the MWAA worker nodes. You can use the following script as a starting point:

```bash
#!/bin/bash
if [[ "${MWAA_AIRFLOW_COMPONENT}" == "worker" ]]; then
  mkdir -p /usr/local/airflow/dbt/libs
  pip3 install  dbt-core==1.7.0 \
   dbt-athena-community==1.7.0 \
    -t /usr/local/airflow/dbt/libs
fi
```

This basically installs DBT and its dependencies into a dedicated folder, irregardless of other Python dependencies specified in requirements.txt.

- You then need to upload to upload this file to S3 deployment bucket and update your MWAA environment to use this file as a startup script. You can follow the [AWS documentation](https://docs.aws.amazon.com/mwaa/latest/userguide/using-startup-script.html) to do this.

- Unfortunately, installing dbt into a separate location would not create the DBT python executable. Hence we need to create that ourselves by creating a Python file in our dags folder `dbt_runner.py`. You can use the following script as a starting point:

```python
from dbt.version import get_installed_version
installed_minor_version = int(get_installed_version().minor)

if installed_minor_version >= 4: # dbt >= 1.4 code path
    from dbt.cli.main import cli
    if __name__ == '__main__':
        sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
        sys.exit(cli())
else:
    from dbt.main import main
    if __name__ == '__main__':
        sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
        sys.exit(main())
```

### How to use in Airflow DAG

Assuming that you are deploying your DBT files to the `dags` folder in your MWAA deployment bucket, you can use the following code to trigger a DBT run


```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
import os
DAG_ID = os.path.basename(__file__).replace(".py", "")
with DAG(dag_id=DAG_ID, schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="run_dbt_bash",
        bash_command="""
export PYTHONPATH=/usr/local/airflow/dbt/libs;        
cp -R /usr/local/airflow/dags/dbt /tmp;
cd /tmp/dbt/dbt-starter-project;
python3 /usr/local/airflow/dags/dbt_runner.py run --project-dir /tmp/dbt/dbt-starter-project/ --profiles-dir ..;
RUN_STATUS=$?;
cat /tmp/dbt/dbt-starter-project/logs/dbt.log; 
exit $RUN_STATUS;
""";
    )

```

This is a very naive approach to running DBT in Airflow, it works for simple use cases however you can easily run into some issues. 

Those issues stem from the fact that MWAA uses [fargate pods](https://docs.aws.amazon.com/mwaa/latest/userguide/what-is-mwaa.html#architecture-mwaa) to run the DBT bash based tasks. This means that multiple tasks can share the same fargate pod.

- Running multiple DBT tasks in parallel might cause issues if they are scheduled in the same pod as they will try to write to the same log file.
As your usage of DBT in Airflow grows, you will need to use a more robust approach to running DBT in Airflow. I will cover this in a future blog post or update this one.

- Disk space issues: The fargate pods used by MWAA have a limited amount of disk space. Depending on the size of the used DBT libraries and other dependencies installed using `requirements.txt`, the pod can end up having less than 8 GB of free disk space left, combine that with frequent DBT runs and you would end up with a full disk. This will cause the MWAA scheduler to either hang or new tasks would silently fail to start.

We can mitigate the above issues by using a temp folder per run and deleting it once the run is complete

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
import os
DAG_ID = os.path.basename(__file__).replace(".py", "")
with DAG(dag_id=DAG_ID, schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="run_dbt_bash",
        bash_command="""        
DIR=$(mktemp -d);        
export PYTHONPATH=/usr/local/airflow/dbt/libs;        
cp -R /usr/local/airflow/dags/dbt $DIR;
cd $DIR/dbt/dbt-starter-project;
python3 /usr/local/airflow/dags/dbt_runner.py run --project-dir $DIR/dbt/dbt-starter-project/ --profiles-dir ..;
RUN_STATUS=$?;
cat $DIR/dbt-starter-project/logs/dbt.log; 
rm -Rf $DIR;
exit $RUN_STATUS;
 """;

    )

```
