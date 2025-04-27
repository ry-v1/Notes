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