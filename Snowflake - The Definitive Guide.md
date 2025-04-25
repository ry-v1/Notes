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