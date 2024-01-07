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

Code is published under this branch [step-1-add-dev-container](https://github.com/ohm-s/dbt-duckdb-sample/tree/step-1-add-dev-container)

To create a devcontainer, we need to create a `.devcontainer` folder in the root of our project. This folder will contain all the configuration files required to set up our devcontainer. We will be using the following files:
- .devcontainer/devcontainer.json
- .devcontainer/dbt/Dockerfile
- .devcontainer/dbt/requirements.txt


{{< tabs "devcontainer" >}}
{{< tab "Devcontainer" >}} 
{{< ghcode "https://github.com/ohm-s/dbt-duckdb-sample/blob/step-1-add-dev-container/.devcontainer/devcontainer.json" >}} 
{{< /tab >}}
{{< tab "Dockerfile" >}} 
{{< ghcode "https://github.com/ohm-s/dbt-duckdb-sample/blob/step-1-add-dev-container/.devcontainer/dbt/Dockerfile" >}} 
{{< /tab >}}
{{< tab "requirements.txt" >}} 
{{< ghcode "https://github.com/ohm-s/dbt-duckdb-sample/blob/step-1-add-dev-container/.devcontainer/dbt/requirements.txt" >}} 
{{< /tab >}}
{{< /tabs >}}  

Once we open the project in VSCode, we will be prompted to reopen the project in a devcontainer. This will build the devcontainer and open the project in the container. This will take a few minutes to complete. Once the container is built, we will have a fully functional DBT environment with DuckDB driver installed.

