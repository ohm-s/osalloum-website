---
author: ["Me"]
title: "How to use DBT core with DuckDB driver - Environment Setup"
description: "A quick guide to set up dbt-core with DuckDB driver on VSCode"
summary: "Setting up DBT core with DuckDB driver on VSCode"
tags: ["dbt", "duckdb", "vscode"]
categories: ["tech-guide"]
series: ["DBT Tutorials"]
ShowToc: true
TocOpen: true
editPost:
    URL: "https://github.com/ohm-s/306-dev-blog/edit/main/content/posts/dbt-duckdb-tutorial/intro.md"
    Text: "Suggest Changes" 
    appendFilePath: false

---

### Why this guide?

#### What is DBT?

DBT(data build tool) core is a python based tool that enables data analysts and engineers to transform data in their datalake/warehouse more effectively. It is a popular tool in the data engineering community and is used by many companies to build their data pipelines.

#### What is DuckDB?

DuckDB is an embeddable SQL OLAP database management system. It is designed to be used as an embedded analytical database for data analytics and data management. DuckDB does not have a standalone server and runs in the same process as the application using it. DuckDB provides extensions to integrate with remote files such as Http, S3 compatible storage and also read a multitude of file formats such as CSV, JSON and notably Parquet.


#### Why use DBT with DuckDB?

DuckDB enables running SQL pipelines on a local machine without the need for a database server. This makes it a very suitable candidate for running DBT locally without the need for a complex setup. However, this does not mean that DuckDB is not suitable for production use cases. It is a very capable database that can be used in production on medium sized datasets.

### IDE Setup

For purposes of this tutorial, we will be using VSCode as our IDE. We will leverage the VSCode devcontainer feature to set up our environment. This will allow us to have a consistent environment across different machines and operating systems.
First off, you need to install the following:
- [VSCode](https://code.visualstudio.com/download)
- [Docker](https://docs.docker.com/get-docker/)  or any docker compatible container runtime, such as [Podman](https://podman.io/getting-started/installation) or Colima
- [VSCode Dev Containers Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

For more details on how to set up VSCode devcontainers, please refer to the [official documentation](https://code.visualstudio.com/docs/devcontainers/tutorial).

### Creating DevContainer

Code is published under this branch [step-1-add-dev-container](https://github.com/ohm-s/dbt_duckdb_sample/tree/step-1-add-dev-container)

To create a devcontainer, we need to create a `.devcontainer` folder in the root of our project. This folder will contain all the configuration files required to set up our devcontainer. We will be using the following files:
- .devcontainer/devcontainer.json
- .devcontainer/dbt/Dockerfile
- .devcontainer/dbt/requirements.txt
- .vscode/settings.json


{{< tabs "devcontainer" >}}
{{< tab "devcontainer.json" >}} 
<br />Root configuration file for the devcontainer: This defines the docker image to be built and the extensions to be installed in the devcontainer.
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-1-add-dev-container/.devcontainer/devcontainer.json" >}} 
{{< /tab >}}
{{< tab "Dockerfile" >}} 
<br /> Docker image definition for the devcontainer
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-1-add-dev-container/.devcontainer/dbt/Dockerfile" >}} 
{{< /tab >}}
{{< tab "requirements.txt" >}} 
<br /> DBT Python libraries 
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-1-add-dev-container/.devcontainer/dbt/requirements.txt" >}} 
{{< /tab >}}
{{< tab "settings.json" >}} 
<br />This is needed to enable sql code auto-formatting to work in the devcontainer.
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-1-add-dev-container/.vscode/settings.json" >}} 
{{< /tab >}}
{{< /tabs >}}  

Once we open the project in VSCode, we will be prompted to reopen the project in a devcontainer. This will build the devcontainer and open the project in the container. This will take a few minutes to complete. Once the container is built, we will have a fully functional DBT environment with all necessary dependencies installed.

### Initializing DBT project

Once the devcontainer is up & running, we can take the next step of creating a new DBT project. We will be using the following command to create a new DBT project

```bash
dbt init dbt_duckdb_sample
```

Select 1 when prompted to create a new DBT project to use the DuckDB profile

Then the command will generate the DBT under project structure under the `dbt_duckdb_sample` folder. And it will generate ` /root/.dbt/profiles.yml` which is DBT (connection profile)[https://docs.getdbt.com/docs/core/connect-data-platform/connection-profiles]

We will copy this file to our project root and track in it in git as part of our project. 

```bash
mkdir profiles
mkdir storage
cp  /root/.dbt/profiles.yml profiles/
```

Then we modify the `profiles.yml` to use relative paths to store the duckdb files under the `storage` folder. 
While we are at it, we'll add our `.gitignore` file 

{{< tabs "profiles" >}}
{{< tab "profiles.yml" >}}
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-2-init-dbt-project/profiles/profiles.yml" >}} 
{{< /tab >}}
{{< tab ".gitignore" >}}
{{< ghcode "https://github.com/ohm-s/dbt_duckdb_sample/blob/step-2-init-dbt-project/.gitignore" >}} 
{{< /tab >}}
{{< /tabs >}}


### Running DBT

Now that we have our DBT project initialized, we can start running DBT commands. We will start by running the following command to check if our DBT project is set up correctly.

```bash
bash-4.2# cd dbt_duckdb_sample
bash-4.2# pwd
/root/dbt_duckdb_sample/dbt_duckdb_sample
bash-4.2# dbt run --profiles-dir ../profiles/
19:27:59  Running with dbt=1.7.0
19:27:59  Registered adapter: duckdb=1.7.0
19:27:59  Found 2 models, 4 tests, 0 sources, 0 exposures, 0 metrics, 391 macros, 0 groups, 0 semantic models
19:27:59  
19:27:59  Concurrency: 1 threads (target='dev')
19:27:59  
19:27:59  1 of 2 START sql table model main.my_first_dbt_model ........................... [RUN]
19:28:00  1 of 2 OK created sql table model main.my_first_dbt_model ...................... [OK in 0.13s]
19:28:00  2 of 2 START sql view model main.my_second_dbt_model ........................... [RUN]
19:28:00  2 of 2 OK created sql view model main.my_second_dbt_model ...................... [OK in 0.08s]
19:28:00  
19:28:00  Finished running 1 table model, 1 view model in 0 hours 0 minutes and 0.40 seconds (0.40s).
19:28:00  
19:28:00  Completed successfully
19:28:00  
19:28:00  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```