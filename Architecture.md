# Cloud Data Platform Architecture

## A tool agnostic approach

```mermaid

flowchart LR;
  
  source_api(REST API) <--> 
    |Call API| extract_n_load

  source_database[(Database)] <-->
    |Read changes| extract_n_load

  source_stream(Streaming) -->
    |Ingest| auto_ingestions          
   
  subgraph Ingestion

    extract_n_load(Extract & Load) -->
      |Load| staging

    auto_ingestions(Auto Ingestion) -->
      |Ingest| staging
      
  end

  subgraph Data Warehouse

    staging(Staging) -->
      |Merge| raw_data

    raw_data(Raw Data) -->
      |Transoform| mart

  end
    
  subgraph Analytics

    mart(Data Mart) --> 
      | Read | prescriptive(Prescriptive Analitycs)

    mart(Data Mart) --> 
      | Read | predictive(Predictive Analytics)

  end
   
```



## A tooling approach

Assumptions made are:
- The team has chosen 
  - [Snowflake](https://www.snowflake.com/) as the cloud data platform for analytics.
  - Apache Airflow [Astronomer Fully Managed](https://www.astronomer.io/)) as the data pipeline orchestrator.
  - [GitLab](https://gitlab.com/) as the version control system and CI/CD tool.
  - to write less code as possible for extracting from the relational databases.
  - Python 3.10 across the different stages of the data pipeline. 
  - to stream events from Kafka and S3
  - [dbt Cloud](https://www.getdbt.com/product/what-is-dbt/) as the tool for implementing transformation and build data marts for analytics.

### Extraction and ingestion

- Databricks can leverage data to Snowflake by calling APIs, reading from relational and NoSQL databases, and is able to scale easily to adjust to and increasing workload.
- [Fivetran](https://www.fivetran.com/) can reliabily leverage data to Snowflake reading from relational database (as per requirement) with the least developing effort and a small administrative effort, as long as the source database has the Change Data Capture or Change Tracking feature enabled.     
- Snowflake can automatically ingest data from Kafka and S3.  

```mermaid

flowchart LR;
  
  source_api(REST API) <--> 
    |Python Notebooks| databricks(((Databricks)))

  source_database[(Database)] <-->
    |*CDC or **CT enabled| fivetran(((Fivetran)))

  source_stream(Streaming) -->
    |Kafka or S3| snowpipe(((Snowpipe)))          
   
  databricks -->
    |Load| Staging

  fivetran -->
    |Load| Staging    
    
  snowpipe -->
    |Load| Staging     
   
```

(*) CDC = Change Data Capture

(**) CT = Change Tracking


### Transformation

- Snowflake provides schedulable and executable Tasks that can run SQL, Javascript, and - in the foreseeable future - Python code. 
- Dbt (data-build-tool) can levarage reliable data models for analytics by using SQL and YAML, for implemeting also schema and data tests and adopting Continous Integration out of the box. 

```mermaid

flowchart LR;

  staging(Staging) -->
    |SQL Javascript Python| snowflake_tasks

  snowflake_tasks(((Snowflake Task))) -->
    |Merge| raw_data

  raw_data(Raw Data) -->
    |SQL Jinja| dbt

  dbt(((data-build-tool))) -->
    |Transoform| mart(Data Mart)
  

```

### Data access

### Data warehousing

### BI-tool for analysts

### Orchestration

