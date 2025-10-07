# \[Part 1 - Admin POV] Databricks SQL: Hands-On Lab

Welcome to the **Administrator POV** section of the Databricks SQL Hands-On Lab!

## Objectives

By the end of this section, you will:

* Understand the responsibilities of a Databricks Workspace Administrator.
* Successfully configure a serverless SQL Warehouse.
* Efficiently manage data assets using catalogs, schemas, and volumes.
* Create streaming tables and optimize queries using materialized views.
* Implement primary and foreign key constraints for data integrity.
* Monitor and optimize SQL queries through query history and profiling.
* Leverage AI to auto-generate documentation of your data assets.
* Manage user access through effective permission settings.

## Target Audience

This guide is tailored specifically for roles responsible for data governance, management, and administration, including:

* Data Architects
* Data Warehousing Architects
* Database Administrators
* Databricks Workspace Administrators

## Lab Outline

We will use a sample dataset containing airline and airport data to demonstrate Databricks SQL’s capabilities from an administrative perspective. The following sections detail each step:

* **Intro:** Navigating Databricks SQL as a Workspace Administrator
* **Step 0:** Configure Serverless SQL Warehouse
* **Step 1:** Create a catalog and define a catalog name
* **Step 2:** Create a schema/database and define schema/database name
* **Step 3:** Create a volume
* **Step 4:** Create streaming tables
* **Step 5:** Create materialized views
* **Step 6:** View query history and query profiles
* **Step 7:** Generate AI-based documentation for data assets
* **Step 8:** Set permissions on data assets

## Let's Get Started

First, we'll dive into the essential role of the Databricks Workspace Administrator and explore the tools and strategies to effectively manage your data!

## Intro: Navigating Databricks SQL as a Workspace Administrator

Once you’re logged into the workspace, Databricks presents a landing page with unified navigation.

The Databricks workspace offers a unified navigation experience, providing easy access to various data engineering, data science, and analytics tools. On the left-side navigation pane, users can find a comprehensive list of features, with a notable focus on SQL capabilities. The SQL section is called **Databricks SQL** and it is a complete data warehouse that includes:

* SQL Editor: create and run SQL queries
* Queries: monitor and organize saved and historical queries
* Dashboards: visualize data and build shareable dashboards
* Genie (natural language querying): interact with data using natural language
* Alerts: create custom alerts based on query thresholds
* Query History: review performance, duration, and resource usage
* SQL Warehouses: compute resources that let's you query and explore data on Databricks


![Databricks SQL Navigation Screenshot](https://data-ai-lakehouse.cloud.databricks.com//files/tables/images/Screenshot_2025_06_04_at_11_26_36_AM.png)

### Pre-Lab Setup Checklist

* [ ] You have a Databricks account with necessary user privileges (you are NOT an admin in this workspace)
* [ ] You can access the Databricks SQL section from the navigation panel (already enabled prior to workshop)
* [ ] Your workspace has Unity Catalog enabled (already enabled prior to workshop)

> **Note for First-Time Users:** Spend a few minutes exploring the layout. Understanding where each SQL tool lives will make this lab much smoother.

## Step 0: Configure SQL Warehouse

As the admin of a Databricks workspace, you are responsible for configuring and monitoring SQL Warehouses. These are the compute resources that execute SQL queries within Databricks SQL.

Follow these steps to inspect or create a SQL Warehouse:

1. From the left-hand navigation pane, click on **SQL Warehouses**
2. You may see a pre-configured SQL Warehouse that is already running. If it is not, click the **play icon** to start it
3. Click on the warehouse name to review its configuration. You may not be able to edit it in this lab, and that's okay, your instructor will demonstrate these steps
4. If you have permission to edit or create a warehouse, click **Edit** or **Create Warehouse** and configure the following settings:

### SQL Warehouse Settings Explained

* **Name**: Choose something descriptive like `Serverless Starter Warehouse`
* **Cluster Size**: We recommend starting with an **2x small** cluster. Larger clusters offer more parallelism and faster response time
* **Auto Stop**: Defines how many idle minutes should pass before the warehouse automatically shuts down to save costs
* **Scaling**: This handles **horizontal scaling**. By default, a warehouse can handle up to 10 concurrent queries. You can enable auto-scaling to support more users without queueing
* **Serverless**: Opt for serverless to get auto-managed compute with faster startup times and automatic scaling
* **Tags**: Add metadata tags (key-value pairs) to track usage and cost by team, project, or department
* **Unity Catalog**: Enable Unity Catalog to manage permissions at the catalog, schema, table, and view levels. It also enables fine-grained audit logging

> **Reminder:** In this lab, your permissions are limited. You may not be able to edit warehouses directly, but these steps are included so you can understand how admins configure compute environments in production settings

### Warehouse Permissions

SQL Warehouses have five permission levels:

* **No Permissions**
* **Can Monitor**
* **Can Use**
* **Can Manage**
* **Can View**

Your ability to interact with warehouses depends on your assigned role.

### Connection Details

The **Connection Details** tab provides credentials and tokens for integrating Databricks SQL with participating apps, tools, SDK, or API

* When needed, you can generate a **personal access token** for secure connections and then...
* configure connection URLs and ports as needed

### Monitoring

The **Monitoring** tab helps you:

* Track the number of active queries
* See how many clusters are allocated
* Review performance metrics in real-time

Now that your SQL Warehouse (compute layer) is configured, you're ready to start ingesting and managing data in your lakehouse!

## Step 1: Create a Catalog and Catalog Name

In Unity Catalog, a **catalog** is the topmost container in the data governance hierarchy. It’s used to logically organize your data across workspaces and manage access control. Catalogs hold schemas, which in turn contain tables, views, functions, and volumes.

In this step, we’ll create a catalog using SQL commands in the Databricks SQL Editor. This is a common administrative task, typically performed by data stewards, administrators, or architects to structure how data is grouped and governed.

> **Note:** In a real-world workspace, you can also create a catalog using the **Catalog Explorer** UI. If you’d like to explore that method, visit the [official Databricks documentation](https://docs.databricks.com/en/catalogs/create-catalog.html#create-a-catalog).

### What You'll Learn

* What a catalog is and why it’s essential in Unity Catalog
* How to dynamically generate a catalog name based on your username
* How to create a catalog using SQL

### Instructions

1. Open the SQL Editor, and toggle on the New SQL Editor
2. Create a new query and **title it "Create Catalog"**
3. Copy and paste the SQL code below
4. Click **Save** to store your query

### Code to Dynamically Create a Catalog Name

```sql
-- 1. Retrieve the current user and extract the part before '@'
WITH current_user_info AS (
  SELECT
    CASE
      WHEN POSITION('@' IN current_user()) > 0 THEN SUBSTRING(current_user(), 1, POSITION('@' IN current_user()) - 1)
      ELSE current_user()
    END AS current_user_no_at
),

-- 2. Replace non-word characters with '_'
sanitized_user AS (
  SELECT
    REGEXP_REPLACE(current_user_no_at, '\\W+', '_') AS sanitized_username
  FROM current_user_info
),

-- 3. Construct the catalog name
catalog_name AS (
  SELECT
    CONCAT('dbsql_demo_', sanitized_username) AS catalogName
  FROM sanitized_user
)

-- 4. Display the resulting catalog name
SELECT * FROM catalog_name;
```

> **Note:** This SQL query helps generate a personalized catalog name using your Databricks username, which is useful in shared environments. Even if you do not have permission to run this in your environment, it’s important to understand how it works.

### Create the Catalog

1. Once you’ve generated your personalized catalog name, under the code where you created the catalog name, copy and paste the code below
2. Replace <catalog> in the command below with the name returned above
3. Run just the code below 
4. Save the query again

```sql
CREATE CATALOG IF NOT EXISTS <catalog>;
```

### Best Practices for Naming Catalogs in a Real-World Situation

* Use lowercase and underscores: `marketing_data`, `finance_reporting`
* Avoid special characters or spaces
* Ensure names are aligned with data domain or department ownership

Once the catalog is created, you can confirm in the catalog explorer. Now, we’ll move to the next layer—schemas (or databases), which live inside the catalog and further organize your data assets.

## Step 2: Create a Schema and Schema Name

In Unity Catalog, a **schema** (also known as a **database**) is the second layer in the data object hierarchy. Schemas reside within catalogs and are used to organize tables, views, functions, and other data objects.

In this step, you'll create a schema using SQL commands in the [Databricks SQL Editor](https://docs.databricks.com/en/sql/sql-editor.html). While you can also create a schema using the **Catalog Explorer**, this lab will focus on using SQL so participants gain hands-on experience writing queries.

> You can also explore how to create schemas using the UI in the [Databricks documentation](https://docs.databricks.com/en/schemas/create-schema.html#create-a-schema).

### What You'll Learn

* How schemas help organize data in Unity Catalog
* How to dynamically generate a unique schema name
* How to create a schema using SQL

### Why Use Dynamic Schema Names?

Just like with catalog names, generating schema names dynamically based on your username ensures that each user has a **unique and isolated environment** and make sharing catalogs easier between team members.

### Instructions

1. Open the SQL Editor
2. Create a new query and **title it "Create Schema"**
3. Copy and paste the SQL code below
4. Click **Save** to store your query

### Code to Dynamically Create a Schema Name

```sql
-- 1. Retrieve the current user and extract the part before '@'
WITH current_user_info AS (
  SELECT
    CASE
      WHEN POSITION('@' IN current_user()) > 0 THEN SUBSTRING(current_user(), 1, POSITION('@' IN current_user()) - 1)
      ELSE current_user()
    END AS current_user_no_at
),

-- 2. Replace non-word characters with '_'
sanitized_user AS (
  SELECT
    REGEXP_REPLACE(current_user_no_at, '\\W+', '_') AS sanitized_username
  FROM current_user_info
),

-- 3. Construct the schema name
db_name AS (
  SELECT
    CONCAT('airlinedata_', sanitized_username) AS dbName
  FROM sanitized_user
)

-- 4. Display the resulting schema name
SELECT dbName FROM db_name;
```

### Create the Schema

1. Once you’ve generated your personalized schema name, under the code where you created the schema name, copy and paste the code below
2. Replace the placeholders in the commands below with your actual schema name you generated above
3. There is a catalog and schema selector. Go ahead and set it to the catalog that you created so that the schema can be created within that catalog
4. Run just the selected text of the code below 
5. Save the query again

```sql
-- 1. Create Schema
CREATE SCHEMA IF NOT EXISTS <schema>;
```

Confirm that your schema has been created by checking the catalog explorer. With your catalog and schema now established, you're ready to start creating storage containers and loading data!

### Step 3: Create A Volume
### 
In Unity Catalog, a **volume** is designed to provide governance over **non-tabular datasets**, such as raw files in structured, semi-structured, or unstructured formats. Volumes act as logical containers for data stored in underlying cloud object storage like AWS S3 or Azure Blob Storage.

> **Important Context for This Lab:**
> In a real-world production environment, users typically ingest or stream data into Databricks from their own cloud storage (e.g., S3 buckets or ADLS). However, to simplify setup and ensure consistent access for the purposes of this lab, especially since not all users have cloud credentials or admin privileges, we will **create a volume** that you’ll use in the next part of this lab.

1. Click each of these and download all three files [airports.csv](https://github.com/sprencode/data-warehousing/blob/main/airports.csv), [flights.csv](https://stanfordhealthcare.sharepoint.com/:x:/r/sites/TDSAnalytics/Shared%20Documents/Data%20Governance/flights.csv?d=wc0c7916e61f34d61a457df608e372bbb&csf=1&web=1&e=oSMfnq) and [lookupcodes.csv](https://github.com/sprencode/data-warehousing/blob/main/lookupcodes.csv) to your computer. Note: right click and open a new tab to get the files to avoid losing your github window

2. Then create a volume in the schema you created and title it **raw_airline_files**. This can be done by navigate to the schema in Catalog and click on the <Create> button and select <Volume>

3. Within that volume called **raw_airline_files** please create a directory called flights, a directory called airports and a directory called lookupcodes. This can be done by navigating to the volume in Catalog and click on the <Create Directory> button.

4. Lastly, upload the three CSV files to each directory you created. (e.g. flights.csv to flights directory; airports.csv to airports directory; and lookupcodes.csv to lookupcodes directory)

You can create regular delta tables from the files in your volume, but for this lab, we will use SQL to extract, transform and load data from files using streaming tables and materialized views.

> **Important Limitation:** You **cannot** use volumes as a location for tables. Volumes are intended for **path-based data access only**. If you want to work with tabular data using Unity Catalog’s capabilities, you should use **tables** instead. We will convert these files

### What You'll Learn

* What volumes are used for in Unity Catalog
* The types of data volumes can support
* Why volumes are different from tables
* How to navigate to a pre-configured volume and upload files

### Volume Purpose Recap

Volumes are often used to:

* Store files in formats like CSV, JSON, Parquet, Avro, images, and more
* Manage data access and auditing using Unity Catalog’s governance features
* Serve as landing zones for unstructured or semi-structured files before transformation

### Summary

Now that the data files are uploaded into your volume, you’ll be able to reference them in future steps as you build streaming tables, materialized views, and more.

Let’s begin working with these files to extract, transform, and load your data using SQL!

## Step 4: Create Streaming Tables

[Streaming tables](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-streaming-table.html#create-streaming-table) are stateful tables designed to continuously ingest and process new data. Because many datasets grow over time, streaming tables are ideal for real-time or near-real-time ingestion use cases.

Streaming tables:

* Ensure each row is processed once
* Support incremental and continuous computation
* Are optimized for freshness and low-latency pipelines

> **Note:** In production, streaming tables usually source data from message queues or cloud object storage. For this lab, we’ll simulate streaming using the static files you uploaded to your shared volume

We’ll organize data following the [Medallion Lakehouse Architecture](https://docs.databricks.com/en/lakehouse/medallion.html#what-is-the-medallion-lakehouse-architecture):

* **Bronze:** Raw ingested data
* **Silver:** Cleansed and conformed data
* **Gold:** Curated business-level data

We’ll start by creating Bronze streaming tables using the CSVs in your volume.

### Instructions

1. Open the SQL Editor
2. **Create a new query** and save it with the appropriate title (e.g., `Bronze_ST`)
3. Copy and paste the following code snippets into the SQL Editor, replacing `<catalog>` and `<schema>` with the names you created in earlier steps
4. In the SQL editor, there is a catalog and schema selector. Go ahead and set it to the catalog and the schema that you created
5. Run each snippet of code 1 at a time!

### Streaming Table: Airports

```sql
-- Create airports streaming table
CREATE OR REFRESH STREAMING TABLE airports_bronze_st AS
SELECT * FROM STREAM read_files('/Volumes/<catalog>/<schema>/raw_airline_files/airports');
```

### Streaming Table: Flights

```sql
-- create flights streaming table
CREATE OR REFRESH STREAMING TABLE flights_bronze_st AS
SELECT * FROM STREAM read_files('/Volumes/<catalog>/<schema>/raw_airline_files/flights');
```

### Streaming Table: Lookupcodes

```sql
-- create lookup codes streaming table
CREATE OR REFRESH STREAMING TABLE lookupcodes_bronze_st AS
SELECT * FROM STREAM read_files('/Volumes/<catalog>/<schema>/raw_airline_files/lookupcodes');
```

Once these tables are created, you’ll have foundational raw data to begin cleansing and transforming in the next steps of the lab.

---

### Other Ways to Ingest Data into Databricks

Databricks supports a variety of ingestion methods depending on your use case and scale. While we're using volumes here for simplicity, these are the more robust ways to bring data into Databricks in production:

* **Add Data UI**: A guided interface for uploading files and connecting to cloud storage
* **SQL Editor or Notebooks**: Manual or scheduled ingestion using SQL or Python
* **Auto Loader**: Efficient, scalable ingestion of new files from cloud storage with minimal setup
* **COPY INTO**: Ideal for SQL users needing idempotent and incremental ingestion from cloud storage
* **Delta Live Tables (DLT)**: Declarative pipelines with built-in monitoring and data quality checks
* **LakeFlow Connect**: Fully managed connectors for enterprise applications, cloud storage, and databases
* **Structured Streaming**: Real-time ingestion from sources like Kafka or Kinesis

> Databricks recommends using **streaming tables** for incremental ingestion via SQL. They combine ease of use with powerful, production-grade performance.

## Step 5: Create Materialized Views

[Materialized views](https://docs.databricks.com/en/views/materialized.html#use-materialized-views-in-databricks-sql) provide a powerful way to process and transform data efficiently in Databricks. They are useful for compliance, corrections, aggregations, and general change data capture (CDC).

Materialized views reduce cost and improve query performance by pre-computing expensive or frequently executed logic. They also make it easy to clean, enrich, and denormalize raw or bronze-layer tables for downstream analytics.

In this step, we’ll create views for the **Silver** and **Gold** layers of the [Medallion Lakehouse Architecture](https://docs.databricks.com/en/lakehouse/medallion.html#what-is-the-medallion-lakehouse-architecture):

* **Silver Layer:** Validated and enriched data
* **Gold Layer:** Aggregated and business-ready data

> **Note:** Your Silver layer could also consist of streaming tables, depending on your data pipeline needs.

---

### Materialized View: Airports Silver Layer

1. **Open the SQL Editor**.
2. **Create a new query** and name it `Airports Silver_MV`.
3. **Paste the following SQL** and make sure the catalog and schema selector are the catalog and schema that you created

```sql

CREATE OR REPLACE MATERIALIZED VIEW airports_silver_mv
(
  IATA STRING,
  City STRING,
  State STRING,
  CONSTRAINT pk_airports PRIMARY KEY (IATA)
) AS
SELECT IATA, City, State
FROM airports_bronze_st;
```

This view extracts and cleans specific columns from the Bronze layer for analysis while incorporating some data modeling with the addition of a primary key

---

### Materialized View: Lookupcodes Silver Layer

1. **Create a new query** and name it `Lookupcodes Silver_MV`.
2. **Paste the following SQL** and make sure the catalog and schema selector are the catalog and schema that you created

```sql

CREATE OR REPLACE MATERIALIZED VIEW lookupcodes_silver_mv
(
  UniqueCode STRING,
  Description STRING,
  CONSTRAINT pk_lookupcodes PRIMARY KEY (UniqueCode)
) AS
SELECT UniqueCode, Description 
FROM lookupcodes_bronze_st
WHERE UniqueCode IS NOT NULL AND Description IS NOT NULL;
```

This view filters out rows with null values to ensure reliable lookup data while incorporating some data modeling with the addition of a primary key

---

### Materialized View: Flights Silver Layer

1. **Create a new query** and name it `Flights Silver_MV`.
2. **Paste the following SQL** and make sure the catalog and schema selector are the catalog and schema that you created

```sql

CREATE OR REPLACE MATERIALIZED VIEW flights_silver_mv
(
  FlightNum,
  Origin,
  Dest,
  Year,
  Month,
  DayofWeek,
  Date,
  UniqueCarrier,
  TailNum,
  DepTime,
  ArrTime,
  ActualElapsedTime,
  Distance,
  IsArrDelayed,
  IsDepDelayed,
  CONSTRAINT fk_flights_unique_carrier FOREIGN KEY (UniqueCarrier) REFERENCES dbsql_demo_pearl_ubaru.airlinedata_pearl_ubaru.lookupcodes_silver_mv(UniqueCode)
) AS
SELECT 
  FlightNum, 
  Origin, 
  Dest, 
  Year, 
  Month, 
  DayofWeek, 
  Date, 
  UniqueCarrier, 
  TailNum, 
  DepTime, 
  ArrTime, 
  ActualElapsedTime, 
  Distance, 
  IsArrDelayed, 
  IsDepDelayed
FROM flights_bronze_st
WHERE FlightNum IS NOT NULL;
```

This view prepares cleansed flight data for reporting and trend analysis, while incorporating some data modeling with the addition of a foreign key

---


### Materialized View: Airports Gold Layer

1. **Create a new query** and name it `Airports Gold_MV`.
2. **Paste the following SQL** and make sure the catalog and schema selector are the catalog and schema that you created.

```sql

CREATE OR REPLACE MATERIALIZED VIEW airports_by_city_mv AS
SELECT City, COUNT(*) AS number_of_airports
FROM airports_silver_mv
GROUP BY City;
```

This Gold layer view provides an aggregated, business-friendly summary by city.

---

Once these views are created, you’ll be ready to explore your cleansed and transformed data in the next steps of the lab.

Now that we have created all of our tables and views by writing SQL queries, these can be monitored using the Query History and Profile.

## Step 6: View Query History and Query Profile

The [Query History](https://docs.databricks.com/en/sql/user/queries/query-history.html#query-history) page shows all SQL queries that have been run using SQL Warehouses. It retains query information for 30 days and is a valuable tool for monitoring, debugging, and optimizing your queries.

### To access Query History:

1. Navigate to the Query History tab in the left-side navigation pane.
2. Click on any query to view its details, including:

   * Execution duration
   * SQL command
   * Number of rows returned
   * I/O performance
3. Click **“See query profile”** to view detailed performance insights such as execution plan, task metrics, and stages.

The [Query Profile](https://docs.databricks.com/en/sql/user/queries/query-profile.html#query-profile) is a visual interface that helps identify performance bottlenecks and optimization opportunities.

### With the Query Profile, you can:

* Visualize each query task and related metrics: time spent, rows processed, memory used
* Pinpoint the slowest parts of the execution plan
* Detect and resolve common SQL inefficiencies like full table scans or skewed joins

Understanding how to read the query history and query profile is essential for monitoring your workload performance and debugging issues quickly.

Let’s now move on and explore our data through the Catalog Explorer!

## Step 7: Create AI-Generated Documentation for Data Assets

As the owner of a data object, or if you have modification permissions, you can use **Catalog Explorer** to view and add [AI-generated comments](https://docs.databricks.com/en/comments/index.html#add-comments-to-data-and-ai-assets) for Unity Catalog–managed assets. These AI-generated comments help automatically document your catalogs, schemas, tables, and even table columns.

Databricks uses a large language model (LLM) to generate these comments based on object metadata like column names, data types, and structure.

### To add AI-generated documentation:

1. Go to the Catalog Explorer tab in the left-hand navigation pane.
2. Navigate to your `<catalog>` and click the **Description**. Then click Generate Comments. This will auto-generate a description for the catalog.
3. Navigate into your `<schema>` and repeat the process by clicking the **Description** button to add a schema-level description.
4. Locate your `flights_bronze_st` table, view the AI Suggested Description and then click the **Accept** button to generate a table description.
5. Above the column headers of `flights_bronze_st`, click the **AI Generate** button again to generate column-level descriptions.
6. Repeat this for all of your newly created streaming tables and views

> **Note:** You can manually edit or override AI-generated comments at any time.

AI-generated documentation is especially helpful for enabling data discovery and governance across your organization.

With your data now documented and discoverable, you're ready to apply access controls and manage permissions across your data assets.

## Step 8: Set Permissions on Data Assets

Databricks makes it easy to manage data access and governance directly within **Catalog Explorer**. As a workspace admin, or any user with appropriate permissions, you can view, assign, and revoke privileges on Unity Catalog–managed objects like catalogs, schemas, volumes, tables, and more.

### What You Can Do in Catalog Explorer:

* Create catalogs, schemas, volumes, tables, views, etc.
* Manage external locations and delta shares
* View and change object ownership
* Grant or revoke access permissions at multiple levels

### In This Step:

We’ll focus on how to grant and manage permissions for the following asset types:

* **Catalog**
* **Schema**
* **Volume**
* **Tables**
* **Views**

### How to Grant or Revoke Permissions:

1. Navigate to Catalog Explorer
2. Select the object (catalog, schema, table, or volume) you want to manage
3. Click the **Permissions** tab
4. Click **Grant** to assign privileges to a user 
5. Choose from a list of permissions (e.g., `SELECT`, `MODIFY`, `USAGE`, `OWN`), depending on the object type
6. Click **Revoke** if you want to remove access

Setting appropriate access controls is critical to managing security, compliance, and data visibility across your organization. Now you're sure your assets are governed and accessible to the right people.

## Conclusion

Congratulations! You've successfully completed the **Admin POV** portion of this Databricks SQL hands-on lab.

At this stage, you have:

* Created your own catalog and schema within Unity Catalog
* Uploaded and staged data using volumes
* Created streaming tables to simulate real-time ingestion
* Built materialized views across the Bronze, Silver, and Gold layers
* Explored query history and query profiles for performance analysis
* Used AI to auto-generate helpful documentation for your data assets
* Applied fine-grained access controls through Catalog Explorer

This foundational work prepares your environment for efficient collaboration, governance, and analytics. Your workspace users now have the structure, data, and permissions they need to begin their own analysis.

Now, it's time to shift perspectives and see how the [**User POV**](https://github.com/thatgirlpearl01/data-warehousing/edit/main/user.md) interacts with the system you just set up.

Let’s move on!
