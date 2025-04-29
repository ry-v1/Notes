# Snowflake - The Definitive Guide

    - Snowsight, the new Snowflake web UI. 

    - After you log in, a client session is maintained indefinitely with continued user activity. After four hours of inactivity, the current session is terminated and you must log in again. The default session timeout policy of four hours can be changed; the minimum configurable idle timeout value for a session policy is five minutes.

    - To get the current role and warehouse
        ```sql
        SELECT CURRENT_ROLE();
        SELECT CURRENT_WAREHOUSE();
        ```
    
    - USE command to set the context for our database.

        ```sql
            USE DATABASE SNOWFLAKE_SAMPLE_DATA;
        ```

#### The Snowflake Architecture

    - The Snowflake hybrid-model architecture is composed of three layers, which are the cloud services layer, the compute layer, and the data storagelayer, along with the three Snowflake caches.
    
    - The Cloud Services Layer:
        - All interactions with data in a Snowflake instance begin in the cloud services layer, also called the global services layer.
        - The Snowflake cloud services layer is a collection of services that coordinate activities such as authentication, access control, and encryption. 
        - It also includes management functions for handling infrastructure and metadata, as well as performing query parsing and optimization, among other features. 
        - The cloud services layer is sometimes referred to as the Snowflake brain because all the various service layer components work together to handle user requests that begin from the time a user requests to log in.
        - The cloud services layer manages data security, including the security for data sharing. The Snowflake cloud services layer runs across multiple availability zones in each cloud provider region and holds the result cache, a cached copy of the executed query results. The metadata required for query optimization or data filtering are also stored in the cloud services layer.
        - you’ll consume only cloud services resources if you use commands on the INFORMATION_SCHEMA tables or certain metadata-only commands such as the SHOW command.
    
    - The Query Processing (Virtual Warehouse) Compute Layer:
        - A Snowflake compute cluster, most often referred to simply as a virtual warehouse, is a dynamic cluster of compute resources consisting of CPU memory and temporary storage. 
        - A running virtual warehouse is required for most SQL queries and all DML operations, including loading and unloading data into tables, as well as updating rows in tables. 
        - Snowflake’s unique architecture allows for separation of storage and compute, which means any virtual warehouse can access the same data as another, without any contention or impact on performance of the other warehouses. This is because each Snowflake virtual warehouse operates independently and does not share compute resources with other virtual warehouses
        - Resizing a Snowflake virtual warehouse is a manual process and can be done even while queries are running because a virtual warehouse does not have to be stopped or suspended to be resized. However, when a Snowflake virtual warehouse is resized, only subsequent queries will make use of the new size. Any queries already running will finish running while any queued queries will run on the newly sized virtual warehouse.
        - The value of the Auto Resume and Auto Suspend times should equal or exceed any regular gaps in your query workload. For example, if you have regular gaps of four minutes between queries, it wouldn’t be advisable to set Auto Suspend for less than four minutes. If you did, your virtual warehouse would be continually suspending and resuming, which could potentially result in higher costs since the minimum credit usage billed is 60 seconds. Therefore, unless you have a good reason for changing the default Auto Suspend and Auto Resume times, it is recommended to leave the default at 10 minutes.

        - create a medium-sized virtual warehouse, with four clusters, that will automatically suspend after five minutes.

        ```sql
            USE ROLE SYSADMIN;
            CREATE WAREHOUSE WH WITH WAREHOUSE_SIZE = MEDIUM
            AUTO_SUSPEND = 300 AUTO_RESUME = true INITIALLY_SUSPENDED = true;
        ```
        - It is a best practice to create a new virtual warehouse in a suspended state. Unless the Snowflake virtual warehouse is created initially in a suspended state, the initial creation of a Snowflake virtual warehouse could take time to provision compute resources.

        - In order to scale up or down, we’ll use the ALTER command:

        ```sql
            USE ROLE SYSADMIN;
            ALTER WAREHOUSE CH2_WH
            SET WAREHOUSE_SIZE = LARGE;
        ```

    - Centralized (Hybrid Columnar) Database Storage Layer
        - Snowflake’s centralized database storage layer holds all data, including structured and semi-structured data.
        - Each Snowflake database consists of one or more schemas, which are logical groupings of database objects such as tables and views.
        - Snowflake automatically organizes stored data into micro-partitions, an optimized, immutable, compressed columnar format which is encrypted using AES-256 encryption. - - - Snowflake optimizes and compresses data to make metadata extraction and query processing easier and more efficient.
        - Snowflake’s data storage layer is sometimes referred to as the remote disk layer. The underlying file system is implemented on Amazon, Microsoft, or Google Cloud.
        - There are two unique features in the storage layer architecture: Time Travel and zero-copy cloning.
        - Zero-copy cloning offers the user a way to snapshot a Snowflake database, schema, or table along with its associated data. There is no additional storage charge until changes are made to the cloned object, because zero-copy data cloning is a metadata-only operation. 
        - Time Travel allows you to restore a previous version of a database, table, or schema.

    - Snowflake Caching
        - Snowflake will use the cached result set if it is still available rather than executing the query you just submitted. 
        - In addition to retrieving the previous query results from a cache, Snowflake supports other caching techniques. There are three Snowflake caching types: the query result cache, the virtual warehouse cache, and the metadata cache.
        - The virtual warehouse cache is located in the compute layer. The result cache is located in the cloud services layer. The metadata storage cache layer is located in the cloud services layer.
        
        - The fastest way to retrieve data from Snowflake is by using the query result cache.
        - The results of a Snowflake query are cached, or persisted, for 24 hours and then purged.
        - Even though the result cache only persists for 24 hours, the clock is reset each time the query is re-executed, up to a maximum of 31 days from the date and time when the query was first executed. After 31 days, or sooner if the underlying data changes, a new result is generated and cached when the query is submitted again.
        - The result cache is fully managed by the Snowflake global cloud services (GCS) layer, and is available across all virtual warehouses since virtual warehouses have access to all data. 
        - The process for retrieving cached results is managed by GCS. However, once the size of the results exceeds a certain threshold, the results are stored in and retrieved from cloud storage.
        - Another unique feature of the query result cache is that it is the only cache that can be disabled by a parameter. 

            ``` sql
            ALTER SESSION SET USE_CACHED_RESULT=FALSE;
            ```

        - The metadata cache is fully managed in the global services layer where the user does have some control over the metadata but no control over the cache. Snowflake collects and manages metadata about tables, micro-partitions, and even clustering. 
        - For tables, Snowflake stores row count, table size in bytes, file references, and table versions. Thus, a running virtual warehouse will not be needed, because the count statistics are kept in the metadata cache when running a SELECT COUNT(*) on a table.
        - The Snowflake metadata repository includes table definitions and references to the micro-partition files for that table. The range of values in terms of MIN and MAX, the NULL count, and the number of distinct values are captured from micro-partitions and stored in Snowflake.
        - Snowflake also stores the total number of micro-partitions and the depth of overlapping micro-partitions to provide information about clustering.
        - The information stored in the metadata cache is used to build thequery execution plan.
        
        - The traditional Snowflake data cache is specific to the virtual warehouse used to process the query. 
        - Running virtual warehouses use SSD storage to store the micro-partitions that are pulled from the centralized database storage layer when a query is processed.
        - The virtual warehouse data cache is limited in size and uses the LRU (Least Recently Used) algorithm.
        - The virtual warehouse cache is sometimes referred to as the raw data cache, the SSD cache, or the data cache. 
        - This cache is dropped once the virtual warehouse is suspended, so you’ll want to consider the trade-off between the credits that will be consumed by keeping a virtual warehouse running and the value from maintaining the cache of data from previous queries to improve performance.

#### Creating and Managing Snowflake Databases
    - In a relational environment, database objects such as tables and views are maintained within databases. 
    - In Snowflake, the database logically groups the data while the schema organizes it. Together, the database and schema comprise the namespace.
    - We can create two main types of databases: permanent (persistent) and transient. 
    - At the time we create a database, the default will be a permanent database, if we don’t specify which of the two types we want to create.
    - Transient databases have a maximum one-day data retention period, aka Time Travel period, and do not have a fail-safe period.
    - The default Time Travel period is one day but can be up to 90 days for permanent databases; or a user could set the Time Travel period to zero days if no Time Travel period is desired.
    - Snowflake’s fail-safe data recovery service provides a seven-day period during which data from permanent databases and database objects may be recoverable by Snowflake. 
    - The fail-safe data recovery period is the seven-day period after the data retention period ends. Unlike Time Travel data, which is accessible by Snowflake users, fail-safe data is recoverable only by Snowflake employees.
    - SQL commands for Snowflake databases
    ```sql
        CREATE DATABASE
        ALTER DATABASE
        DROP DATABASE
        SHOW DATABASES
    ```
    - The SNOWFLAKE database is owned by Snowflake Inc. and is a system-defined, read-only, shared database which provides object metadata and usage metrics about your account.
    - you can store transient tables within a permanent database but you cannot store permanent tables within a transient database.
    - There is no limit to the number of database objects, schemas, and databases that can be created within a Snowflake account.
    - For regular schemas, the object owner role can grant object access to other roles and can also grant those roles the ability to manage grants for the object. However, in managed access schemas, object owners are unable to issue grant privileges. Instead, only the schema owner or a role with the MANAGE GRANTS privilege assigned to it can manage the grant privileges.
    - Two database schemas, are included in every database that is created: INFORMATION_SCHEMA and PUBLIC. 
    - The PUBLIC schema is the default schema and can be used to create any other objects, whereas the INFORMATION_SCHEMA is a special schema for the system that contains views and table functions which provide access to the metadata for the database and account.

    - The INFORMATION_SCHEMA, also known as the Data Dictionary, includes metadata information about the objects within the database as well as account-level objects such as roles.
    - More than 20 system-defined views are included in every INFORMATION_SCHEMA. These views can be divided into two categories: account views and database views.
    - INFORMATION_SCHEMA account views include the following:
        - APPLICABLE_ROLES : Displays one row for each role grant
        - DATABASES : Displays one row for each database defined in your account
        - ENABLED_ROLES : Displays one row for each currently enabled role in the session
        - INFORMATION_SCHEMA_CATALOG_NAME : Displays the name of the database in which the INFORMATION_SCHEMA resides
        - LOAD_HISTORY : Displays one row for each file loaded into tables using the COPY INTO <table> command, and returns history for all data loaded in the past 14 days except for data loaded using Snowpipe
        - REPLICATION_DATABASES : Displays one row for each primary and secondary database (i.e., a database for which replication has been enabled) in your organization
    
    - INFORMATION_SCHEMA database views include the following views:
        - COLUMNS : Displays one row for each column in the tables defined in the specified (or current) database.
        - EXTERNAL_TABLES : Displays one row for each external table in the specified (or current) database.
        - FILE_FORMATS : Displays one row for each file format defined in the specified (or current) database.
        - FUNCTIONS : Displays one row for each UDF or external function defined in the specified (or current) database.
        - OBJECT_PRIVILEGES : Displays one row for each access privilege granted for all objects defined in your account.
        - PIPES : Displays one row for each pipe defined in the specified (or current) database.
        - PROCEDURES : Displays one row for each stored procedure defined for the specified (or current) database.
        - REFERENTIAL_CONSTRAINTS : Displays one row for each referential integrity constraint defined in the specified (or current) database.
        - SCHEMATA : Displays one row for each schema in the specified (or current) database.
        - SEQUENCES : Displays one row for each sequence defined in the specified (or current) database.
        - STAGES : Displays one row for each stage defined in the specified (or current) database.
        - TABLE_CONSTRAINTS : Displays one row for each referential integrity constraint defined for the tables in the specified (or current) database.
        - TABLE_PRIVILEGES : Displays one row for each table privilege that has been granted to each role in the specified (or current) database.
        - TABLE_STORAGE_METRICS : Displays table-level storage utilization information, includes table metadata, and displays the number of storage types billed for each table. Rows are maintained in this view until the corresponding tables are no longer billed for any storage, regardless of various states that the data in the tables may be in (i.e., active, Time Travel, fail-safe, or retained for clones).
        - TABLES : Displays one row for each table and view in the specified (or current) database.
        - USAGE_PRIVILEGES : Displays one row for each privilege defined for sequences in the specified (or current) database.
        - VIEWS : Displays one row for each view in the specified (or current) database.

    - The SNOWFLAKE database, viewable by the ACCOUNTADMIN by default, includes an ACCOUNT_USAGE schema.
    - The SNOWFLAKE database ACCOUNT_USAGE schema includes records for dropped objects whereas the INFORMATION_SCHEMA does not.
    - The ACCOUNT_USAGE schema has a longer retention time for historical usage data. Whereas the INFORMATION_SCHEMA has data available ranging from seven days to six months, the ACCOUNT_USAGE view retains historical data for one year.
    - Most views in the INFORMATION_SCHEMA have no latency, but the latency time for ACCOUNT_USAGE could range from 45 minutes to three hours. Specifically, for the INFORMATION_SCHEMA, there may be a one- to two-hour delay in updating storage-related statistics for ACTIVE_BYTES, TIME_TRAVEL_BYTES, FAILSAFE_BYTES, and RETAINED_FOR_CLONE_BYTES.

    - All Snowflake data is stored in tables. In addition to permanent and transient tables, it is also possible to create hybrid, temporary, and external tables. 
    - Snowflake hybrid tables support the new Unistore workload. 
    - Snowflake temporary tables only exist within the session in which they were created and are frequently used for storing transitory data such as ETL data. 
    - Snowflake external tables give you the ability to directly process or query your data that exists elsewhere without ingesting it into Snowflake, including data that lives in a data lake.
    - Materialized tables allow users to declaratively specify the pipelines where transformations can occur. Snowflake then handles the incremental refresh to materialize the data.
    - TRUNCATE and DELETE are different in that TRUNCATE also clears table load history metadata while DELETE retains the metadata.

    - Transient tables are designed for transitory data that needs to be maintained beyond a session but doesn’t need the same level of data recovery as permanent tables. 
    - As a result, the data storage costs for a transient table would be less than for a permanent table. 
    - One of the biggest differences between transient tables and permanent tables is that the fail-safe service is not provided for transient tables.
    - It isn’t possible to change a permanent table to a transient table by using the ALTER command, because the TRANSIENT property is set at table creation time. Likewise, a transient table cannot be converted to a permanent table.
    
    - A temporary table, as well as the data within it, is no longer accessible once the session ends. During the time a temporary table exists, it does count toward storage costs; therefore, it is a good practice to drop a temporary table once you no longer need it.
    - Temporary tables exist only within the session in which they are created. This means they are not available to other users and cannot be cloned.

    - Views are of two types: materialized and nonmaterialized. Whenever the term view is mentioned and the type is not specified, it is understood that it is a nonmaterialized view.
    - A view is considered to be a virtual table created by a query expression.
    - Views can provide even more security by creating a specific secure view of either a nonmaterialized or materialized view. 