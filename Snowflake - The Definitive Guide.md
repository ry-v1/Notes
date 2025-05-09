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
    - Materialized views are generally used to aggregate as well as filter data so that the results of resource-intensive operations can be stored in a materialized view for improved data performance. The performance improvement is especially good when that same query is used frequently.
    - Snowflake uses a background service to automatically update materialized views. As a result, data accessed through materialized views is always current, regardless of the amount of DML that has been performed on the base table. 
    - Snowflake will update the materialized view or use the up-to-date portions of the materialized view and retrieve newer data from the base table if a query is run before the materialized view is up to date.
    - As a general rule, it is best to use a nonmaterialized view when the results of the view change frequently and the query isn’t so complex and expensive to rerun. Regular views do incur compute costs but not storage costs. The compute cost to refresh the view and the storage cost will need to be weighed against the benefits of a materialized view when the results of a view change often.
    - A materialized view can query only a single table, and joins are not supported.

    - Stages are Snowflake objects that point to a storage location, either internal to Snowflake or on external cloud storage. 
    - Internal stage objects can be either named stages or a user or table stage. The temporary keyword can be used to create a session-based named stage object.
    - Table stages are useful if multiple users need to access the files and those files only need to be copied into a single table, whereas a user stage is best when the files only need to be accessed by one user but will need to be copied into multiple tables.
    - User stages and table stages, both of which are types of internal stages, are automatically provided for each Snowflake account.
    - Internal named stages are database objects, which means they can be used not just by one user but by any user who has been granted a role with the appropriate privileges.
    - When using stages, we can use file formats to store all the format information we need for loading data from files to tables. The default file format is CSV. However, you can create file formats for other formats such as JSON, Avro, ORC, Parquet, and XML.

    - If you need to perform a database operation such as SELECT, DELETE, or CREATE, you’ll need to use a stored procedure. 
    - If you want to use a function as part of the SQL statement or expression, or if your output needs to include a value for every input row, you’ll want to use a Snowflake UDF.
    - Secure UDFs are the same as nonsecure UDFs, except that they hide the DDL from the consumer ofthe UDF. Secure UDFs do have limitations on performance functionality due to some optimizations being bypassed. Thus, data privacy versus performance is the consideration because only secure UDFs can be shared.

    - Pipes are objects that contain a COPY command that is used by Snowpipe. Snowpipe is used for continuous, serverless loading of data into a Snowflake target table. 
    - Snowflake streams, also known as change data capture (CDC), keep track of certain changes made to a table including inserts, updates, and deletes. 
    - Snowflake streams have many useful purposes, including recording changes made in a staging table which are used to update another table.
    - A sequence object is used to generate unique numbers. Often, sequences are used as surrogate keys for primary key values.

    - A Snowflake stream works like a pointer that keeps track of the status of a DML operation on a defined table. 
    - A Snowflake table stream creates a change table that shows what has changed, at a row level, between two transactional points in time. 
    - Streams are like processing queues and can be queried just like a table. 
    - Table streams make it easy to grab the new data in a table so that one can have more efficient processing.
    - Streams do that by taking a snapshot of all the rows in a table at a point in time and only storing an offset for the source table.
    - The METADATA$ACTION column tells us whether the row was inserted or deleted. If the row is updated, the METADATA$ISUPDATE column will be TRUE. And lastly, there is a unique hash key for the METADATA$ROW_ID column.

    - Tasks run on a schedule which is defined at the time a task is created. Alternatively, you can establish task dependencies whereby a task can be triggered by a predecessor task. There is no event source that can trigger a task; a task that is not triggered by a predecessor task must be run on a schedule.
    - Each individual task is limited to a single predecessor task but can have up to 100 child tasks. A task tree is limited to 1,000 tasks in total, including the root task.
    - The EXECUTE TASK command manually triggers a single run of a scheduled task that is either a standalone task or the root task in a task tree. If it is necessary to recursively resume all dependent tasks tied to a root task, be sure to use the SYSTEM$TASK_DEPENDENTS_ENABLE function, rather than resuming each task individually by using the ALTER TASK RESUME command.
    - Three Snowflake task functions can be used to retrieve information about tasks:
        - SYSTEM$CURRENT_USER_TASK_NAME : This function returns the name of the task currently executing when invoked from the statement or stored procedure defined by the task.
        - TASK_HISTORY : This function returns task activity within the last seven days or the next scheduled execution within the next eight days.
        - TASK_DEPENDENTS : This is a table function that returns the list of child tasks for a given root task.

    - Data Definition Language (DDL) commands: create, modify, and delete database structures.
        - CREATE
        - ALTER
        - TRUNCATE
        - RENAME
        - DROP
        - DESCRIBE
        - SHOW
        - USE
        - SET/UNSET
        - COMMENT
    - Data Control Language (DCL) commands: enable access control.
        - GRANT
        - REVOKE
    - Data Manipulation Language (DML) commands: manipulate the data.
        - INSERT
        - MERGE
        - UPDATE
        - DELETE
        - COPY INTO
        - PUT
        - GET
        - LIST
        - VALIDATE
        - REMOVE
    - Transaction Control Language (TCL) commands: manage transaction blocks.
        - BEGIN
        - COMMIT
        - ROLLBACK
        - CREATE
    - Data Query Language (DQL) command: either a statement or a clause to retrieve data that meets the criteria specified in the SELECT command.
        - SELECT

    - Snowflake SQL queries begin with either the WITH clause or the SELECT command.
    - The WITH clause, an optional clause that precedes the SELECT statement, is used to define common table expressions (CTEs) which are referenced in the FROM clause.


        ----------------------------------------------
        Query syntax  | Query clause | Comments
        ----------------------------------------------
        WITH | | Optional clause that precedes the body of the SELECT statement
        TOP<n> | | Contains the maximum number of rows returned, recommended to include ORDER BY
        FROM AT | BEFORE, CHANGES, CONNECT BY, JOIN, MATCH_RECOGNIZE, PIVOT or UNPIVOT, SAMPLE or TABLESAMPLE_VALUE | Specifies the tables, views, or table functions to use in a SELECT statement
        WHERE | | Specifies a condition that matches a subset of rows; can filter the result of the FROM clause; can specify which rows to operate on in an UPDATE, MERGE, or DELETE
        GROUP BY | GROUP BY CUBE, GROUP BY GROUPING SETS, GROUP BY ROLLUP, HAVING | Groups rows with the same group-by-item expressions and computes aggregate functions for resultant group; can be a column name, a number referencing a position in the SELECT list, or expression
        QUALIFY | | Filters the results of window functions
        ORDER BY | | Specifies an ordering of the rows of the result table from a SELECT list
        LIMIT/FETCH | |  Constrains the maximum number of rows returned; recommended to include ORDER BY
    
    - An uncorrelated subquery is an independent query, one in which the value returned doesn’t depend on any columns of the outer query. An uncorrelated subquery returns a single result that is used by the outer query only once.

    - A correlated subquery references one or more external columns. A correlated subquery is evaluated on each row of the outer query table and returns one result per row that is evaluated.

    - Whenever the same names exist for a CTE and a table or view, the CTE will take precedence. Therefore, it is recommended to always choose a unique name for your CTEs.

    - Arithmetic operators : +, –, * , /, and %
    - Comparison operators : 
        - Equal (=)
        - Not equal (!= or <>)
        - Less than (<)
        - Less than or equal (<=)
        - Greater than (>)
        - Greater than or equal (>=)
    - Logical operators : NOT, AND, OR
    - Subquery operators : [NOT] EXISTS, ANY or ALL, and [NOT] IN
    - Set operators : INTERSECT, MINUS or EXCEPT, UNION, and UNION ALL - order of preference is INTERSECT as the highest precedence, followed by EXCEPT, MINUS, and UNION, and finally UNION ALL as the lowest precedence.

    - The Snowflake system will cancel long-running queries. The default duration for long-running queries is two days, but the STATEMENT_TIMEOUT_IN_SECONDS duration value can always be set at an account, session, object, or virtual warehouse level.

    Fixed-point number data types | Precision | Comments
    NUMBER | Optional (38, 0) | Numbers up to 38 digits; maximum scale is 37
    DECIMAL, NUMERIC | Optional (38,0) | Synonymous with NUMBER
    INT, INTEGER, BIGINT, SMALLINT, TINYINT, BYTEINT | Cannot be specified; always (38,0) | Possible values: -99999999999999999999999999999999999999 to +99999999999999999999999999999999999999 (inclusive)

    Floating-point number data types | Comments
    FLOAT, FLOAT4, FLOAT8 | Approximately 15 digits Values range from approximately 10-308 to 10+308
    DOUBLE, DOUBLE PRECISION, REAL | Approximately 15 digits Synonymous with FLOAT

    Text string data types | Parameters | Comments
    VARCHAR Optional parameter | (N), max number of characters | Holds Unicode characters; no performance difference between using full-length VARCHAR (16,777,216) or a smaller length
    CHAR, CHARACTERS | | Synonymous with VARCHAR; length is CHAR(1) if not specified
    STRING, TEXT | | Synonymous with VARCHAR

    Binary string data types | Comments
    BINARY | Has no notion of Unicode characters, so length is always measured in bytes; if length is not specified, the default is 8 MB (the maximum length)
    VARBINARY | Synonymous with BINARY

    Date and time data types | Default mapping | Comments
    DATE | | Single DATE type; most common date forms are accepted; all accepted timestamps are valid inputs with TIME truncated; the associated time is assumed to be midnight
    DATETIME | | Alias for TIMESTAMP_NTZ
    TIME | | Single TIME type in the form HH:MI:SS, internally stored as wall clock time; time zones not taken into consideration
    TIMESTAMP | Default is TIMESTAMP_NTZ | User-specified alias of one of the three TIMESTAMP_ variations
    TIMESTAMP_LTZ | | Internally UTC time with a specified precision; TIMESTAMP with local time zone
    TIMESTAMP_NTZ | | Internally wall clock time; TIMESTAMP without time zone
    TIMESTAMP_TZ | | Internally UTC time with a time zone offset; TIMESTAMP with time zone

    Semi-structured data types | Characteristics | Comments
    VARIANT | Can store OBJECT and ARRAY | Stores values of any other type, up to a maximum of 16 MB uncompressed; internally stored in compressed columnar binary representation
    OBJECT | | Represents collections of key-value pairs with the key as a nonempty string and the value of VARIANT type
    ARRAY | | Represents arrays of arbitrary size whose index is a non-negative integer and values have VARIANT type

    - Using Snowflake, storing and granting access to unstructured data can be done in three different ways: stage file URLs, scoped URLs, or presigned URLs.

    - A stage file URL is used to create a permanent URL to a file on a Snowflake stage and is used most frequently for custom applications.
    
    - A scoped URL is frequently used for custom applications; especially in situations where access to the data will be given to other accounts using the data share functionality or when ad hoc analysis is performed internally using Snowsight.
    - Access to files in a stage using scoped URL access is achieved in one of two ways. One way is for a Snowflake user to click a scoped URL in the results table in Snowsight. The other way is to send the scoped URL in a request which results in Snowflake authenticating the user, verifying the scoped URL has not expired, and then redirecting the user to the staged file in the cloud storage service.

    - A presigned URL is most often used for business intelligence applications or reporting tools that need to display unstructured file contents for open files.

    - Snowflake built-in functions include scalar, aggregate, window, table, and system functions.

    - Discretionary access control (DAC) is a security model in which each object has an owner who has control over that object. 
    - Role-based access control (RBAC) is an approach in which access privileges are assigned to roles and roles are then assigned to one or more users.
    - Securable object : Entity such as a database or table. Access to a securable object is denied unless specifically granted.
    - Role : Receives privileges to access and perform operations on an object or to create or alter the access control policies themselves. Roles are assigned to users. Roles can,also be assigned to other roles, creating a role hierarchy.
    - Privileges : Inherent, assigned, or inherited access to an object.
    - User : A person, service account, or program that Snowflake recognizes.

    - The account administrator (ACCOUNTADMIN) is at the top level of system-defined account roles and can view and operate on all objects in the account, with one exception. 
    - The account administrator will have no access to an object when the object is created by a custom role without an assigned system-defined role.
    - The ACCOUNTADMIN role can stop any running SQL statements and can view and manage Snowflake billing. 
    - Privileges for resource monitors are unique to the ACCOUNTADMIN role. None of the inherent privileges that come with the resource monitor privileges come with the GRANT option, but the ACCOUNTADMIN can assign the ALTER RESOURCE MONITOR privilege to another role.

    - The security administrator (SECURITYADMIN) is inherently given the MANAGE GRANTS privilege and also inherits all the privileges of the USERADMIN role. The user administrator (USERADMIN) is responsible for creating and managing users and roles.
    
    - The system administrator (SYSADMIN) role is a system-defined role with privileges to create virtual warehouses, databases, and other objects in the Snowflake account. The most common roles created by the USERADMIN are assigned to the SYSADMIN role, thus enabling the system or account administrator to manage any of the objects created by custom roles.
    
    - The PUBLIC role is automatically granted to every role and every user in the Snowflake account. Just like any other role, the PUBLIC role can own securable objects.

    - The most important benefit of batch processing is that it’s typically less expensive than the other types of data processing. It’s less expensive because compute resources are only used when the processing is executed, and since batch processes execute less frequently than near–real-time processing, the overall cost is less. The trade-off, though, is data freshness.

    - Continuous loading has either a small state or no state at all and typically involves relatively simple transformations. Continuous loading is most often used when we need fresh, near–real-time data without needing to know what happened in the last two or three seconds.

    - Snowflake stages are temporary storage spaces used as an intermediate step to lead files to Snowflake tables or to unload data from Snowflake tables into files. 
    - There are two main types of stages: internal and external. 
    
    - For external stages, the files are stored in an external location such as an S3 bucket, and they are referenced by the external stage.
    
    - Internal stage types include internal named stages, user stages, and table stages.
    
    - Internal named stages are database objects; thus, they can be used by any user who has been granted a role with the appropriate privileges.
    - SQL statements in which a named stage is referenced will need the @ symbol along with the name of the stage.
    - To list named stages, you can run the LIST @<stage name> statement.

    - Each Snowflake user has a stage for storing files which is accessible only by that user. A user stage is not a separate database object, and it cannot be altered or dropped.
    - The data in a user stage is accessible only to the specific user who executed the SQL commands. SQL statements in which a user stage is referenced will need the @~ symbols. The LIST @~ SQL statement can be used to list user stages.

    - A table stage is not a separate database object; rather, it is an implicit stage tied to the table itself. 
    - Just like user stages, a table stage cannot be altered or dropped. 
    - Additionally, the data in table stages is accessible only to those Snowflake roles which have been granted the privileges to read from the table. 
    - SQL statements in which a table stage is referenced will need the @% symbols as well as the name of the table. 
    - LIST @%<name of table> is the statement you can use to list table stages.

    - Five different ways to load data into Snowflake. 
        - The first way, is to insert data into tables via SQL command statements in the Snowflake worksheets. 
        - upload files directly in the web UI. 
        - SnowSQL CLI, where we’ll use COPY INTO and PUT commands. 
        - data pipelines and third-party ETL and ELT tools.

    - ETL can be an ideal solution when the destination requires a specific data format that is different from the pipeline definition.
    - ELT approach is often the preferred approach when the destination is a cloud native data warehouse.

    - When using Snowflake’s Snowpipe, there are two different mechanisms for detecting when the staged files are available: automate Snowpipe using cloud messaging and call Snowpipe REST endpoints.
    - As a rule of thumb, the optimal file size for loading into Snowpipe is 100 to 250 MB of compressed data. Try to stage the data within 60-second intervals if the data arrives continuously. Remember, it is possible to create a Snowpipe in such a way that latency can be decreased and throughput can be significantly increased, but the architectural design needs to be weighed against the increased Snowpipe costs that occur as more frequent file ingestions are triggered.
    - There are a few different considerations when deciding between the two Snowpipe methods of auto-ingest or REST API. In situations where files arrive continuously, you can use Snowflake’s auto-ingest feature to create an event notification. Snowpipe auto-ingest is the more scalable approach. REST API is the better option for use cases in which data arrives randomly and/or if preprocessing needs require using an ETL or ELT tool, or in situations in which an external stage is unavailable.