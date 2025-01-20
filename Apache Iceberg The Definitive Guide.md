# Apache Iceberg The Definitive Guide

## Introduction to Apache Iceberg

#### OnLine Transaction Processing (OLTP)

    - Transactional systems are focused on inserting, updating, and reading a small subset of rows in a table, so storing the data in a row-based format is ideal.
    - RDBMSs are used for this OLTP type of data processing category. 
    - Examples of OLTP-optimized RDBMSs are PostgreSQL, MySQL, and Microsoft SQL Server.

#### OnLine Analytical Processing (OLAP)

    - OLTP systems are not designed to deal with complex aggregate queries involving a large number of historical records. 
    - These workloads are known as online analytical processing (OLAP) workloads.

#### Foundational Components of a System Designed for OLAP Workloads

    - Storage
        - Options for storage 
            - Local filesystem on direct-attached storage (DAS)
            - Distributed filesystem on a set of nodes that you operate, such as the Hadoop Distributed File System (HDFS)
            - Object storage provided as a service by cloud providers, such as Amazon Simple Storage Service(Amazon S3)
        - Types of storage 
            - row-oriented databases 
            - columnar databases
            - Mix of the row-oriented and columnar

    - File format
        - File format impacts things such as the compression of the files, the data structure, and the performance of a given workload.       
        - High-level categories
            - structured (CSV), semistructured (JSON), and unstructured (text files).      
        - In the structured and semistructured categories, file formats can be row oriented or column oriented (columnar).   
        - Row-oriented file formats store all the columns of a given row together 
            - Examples of row-oriented file formats are comma-separated values (CSV) and Apache Avro. 
        - Column-oriented file formats store all the rows of a given column together. 
            - Examples of columnar file formats are Apache Parquet and Apache ORC.

    - Table format
        - The table format acts like a metadata layer on top of the file format and is responsible for specifying how the datafiles should be laid out in storage.
        - Goal of a table format is to abstract the complexity of the physical data structure and facilitate capabilities such as Data Manipulation Language (DML) operations (e.g., doing inserts, updates, and deletes) and changing a table's schema.

    - Storage engine
        - Storage engines handle some of the critical tasks, such as physical optimization of the data, index maintenance, and getting rid of old data.

    - Catalog
        - The catalog is the central location where compute engines and users can go to find out about the existence of a table
        - Table name, table schema, and where the table data is stored on the storage system.

    - Compute engine
        - A compute engine's role is to run user workloads to process the data. 
        - Depending on the volume of data, computation load, and type of workload, you can utilize one or more compute engines for this task.
        - Distributed compute engine in a processing paradigm called massively parallel processing (MPP). 
        - Examples of MPP-based compute engines are Apache Spark, Snowflake, and Dremio.

#### Data Lake

    - Data lake has the ability to leverage different compute engines for different workloads.
    - In data lakes, there isn’t really any service that fulfills the needs of the storage engine function. 
    - Generally, the compute engine decides how to write the data, and then the data is usually never revisited and optimized, unless entire tables or partitions are rewritten, which is usually done on an ad hoc basis.
    - Pros
        - Lower cost
        - Stores data in open formats
        - Handles unstructured data
        - Supports ML use cases
    - Cons
        - Performance
        - Lack of ACID guarantees
        - Lots of configuration required

#### The Data Lakehouse

    - The data lakehouse architecture decouples the storage and compute from data lakes and brings in mechanisms that allow for more data warehouse-like functionality (ACID transactions, better performance, consistency, etc.).
    - Data lakehouse is the table format providing a metadata/abstraction layer between the engine and storage for them to interact more intelligently.
    - Table formats create an abstraction layer on top of file storage that enables better consistency, performance, and ACID guarantees when working with data directly on data lake storage.
    - A table format is a method of structuring a dataset's files to present them as a unified “table.”
    - Flaw that led to challenges with the Hive table format was that the definition of the table was based on the contents of directories, not on the individual datafiles.
    - Modern table formats took this approach of defining tables as a canonical list of files, providing metadata for engines informing which files make up the table, not which directories. This more granular approach to defining “what is a table” unlocked the door to features such as ACID transactions, time travel, and more.

#### The Apache Iceberg Architecture

    -This metadata tree breaks down the metadata of the table into four components:
        - Manifest file - A list of datafiles, containing each datafile's location/path and key metadata about those datafiles, which allows for creating more efficient execution plans.
        - Manifest list - Files that define a single snapshot of the table as a list of manifest files along with stats on those manifests that allow for creating more efficient execution plans.
        - Metadata file - Files that define a table's structure, including its schema, partitioning scheme, and a listing of snapshots.
        - Catalog - Tracks the table location (similar to the Hive Metastore), but instead of containing a mapping of table name -> set of directories, it contains a mapping of table name -> location of the table's most recent metadata file. Several tools, including a Hive Metastore, can be used as a catalog.

#### Key Features of Apache Iceberg
    - ACID transactions
        - Apache Iceberg uses optimistic concurrency control to enable ACID guarantees, even when you have transactions being handled by multiple readers and writers.
        - Optimistic concurrency assumes transactions won't conflict and checks for conflicts only when necessary, aiming to minimize locking and improve performance.
        - This way, you can run transactions on your data lakehouse that either commit or fail and nothing in between.
        - A pessimistic concurrency model, which uses locks to prevent conflicts between transactions, assuming conflicts are likely to occur, was unavailable in Apache Iceberg at this time.
        - Concurrency guarantees are handled by the catalog, as it is typically a mechanism that has built-in ACID guarantees.
    - Partition evolution
        - With Apache Iceberg you can update how the table is partitioned at any time without the need to rewrite the table and all its data. 
        - Since partitioning has everything to do with the metadata, the operations needed to make this change to your table's structure are quick and cheap.
    - Hidden partitioning
        - In Iceberg, partitioning occurs in two parts: 
            - the column, which physical partitioning should be based on 
            - an optional transform to that value including functions such as bucket, truncate, year, month, day, and hour. 
            - The ability to apply a transform eliminates the need to create new columns just for partitioning.
    - Row-level table operations
        - You can optimize the table's row-level update patterns to take one of two forms: copy-on-write (COW) or merge-on-read (MOR). 
        - When using COW, for a change of any row in a given datafile, the entire file is rewritten (with the row-level changemade in the new file) even if a single record in it is updated. 
        - When using MOR, for any row-level updates, only a new file that contains the changes to the affected row that is reconciled on reads is written. This gives flexibility to speed up heavy update and delete workloads.
    - Time travel 
        - Apache Iceberg provides immutable snapshots, so the information for the table's historical state is accessible, allowing you to run queries on the state of the table at a given point in time in the past
    - Version rollback
        - Not only does Iceberg's snapshot isolation allow you to query the data as it is, but it also reverts the table's current state to any of those previous snapshots.
    - Schema evolution
        - Tables change, whether that means adding/removing a column, renaming a column, or changing a column's data type.


## The Architecture of Apache Iceberg

    - The Data Layer
        - stores the actual data of the table and is primarily made up of the datafiles themselves, although delete files are also included. 
        - provides the user with the data needed for their query.
        - some exceptions where structures in the metadata layer can provide a result (e.g., get me the max value for column X)
        - Datafiles 
            - store the data itself. 
            - Apache Iceberg is file format agnostic.
            - Parquet might be used for large-scale online analytical processing (OLAP) analytics. 
            - Avro might be used for low-latency streaming analytics tables.
        - Delete Files
            - track which records in the dataset have been deleted.
            - Positional delete files denote what rows have been logically deleted, by identifying the exact position in the table where the row is located.
            - Equality delete files denote what rows have been logically deleted, by identifying the row by the values of one or more of the fields for the row.

    - The Metadata Layer
        - contains all the metadata files
        - Manifest Files
            - Manifest files keep track of files in the data layer (i.e., datafiles and delete files) as well as additional details and statistics about each file, such as the minimum and maximum values for a datafile's columns.
            - Manifest files are the files that do this tracking at the leaf level of the metadata tree.
            - Each manifest file keeps track of a subset of the datafiles. 
            - They contain information such as details about partition membership, record counts, and lower and upper bounds of columns that are used to improve efficiency and performance while reading the data from these datafiles.
            - In Iceberg, manifest files are in Avro format.
        - Manifest Lists
            - A manifest list is a snapshot of an Iceberg table at a given point in time.
            - A manifest list contains an array of structs, with each struct keeping track of a single manifest file.
        - Metadata Files
            - Manifest lists are tracked by metadata files.
            - Each time a change is made to an Iceberg table, a new metadata file is created and is registered as the latest version of the metadata file atomically via the catalog.
        - Puffin Files
            - stores statistics and indexes about the data in the table that improve the performance of an even broader range of queries.
            - The file contains sets of arbitrary byte sequences called blobs, along with the associated metadata required to analyze the blobs.

    - The Catalog
        - The central place where you go to find the current location of the current metadata pointer is the Iceberg catalog.

## Lifecycle of Write and Read Queries

    - How a query engine interacts with the iceberg components for reads and writes
        - Catalog layer
            - catalog is the first component that a query engine interacts with. 
            - for reads, the engine reaches out to the catalog to learn about the current state of the table
            - for writes, the catalog is used to adhere to the schema defined and to know about Metadata layer the table's partitioning scheme.

        - Metadata layer
            - Each time a query engine writes something to an Iceberg table, a new metadata file is created atomically and is defined as the latest version of the metadata file.
            - during read operations, engines will always see the latest version of the table. 
            - Query engines interact with the manifest lists to get information about partition specifications that help them skip the nonrequired manifest files for faster performance.
            - information from the manifest files, such as upper and lower bounds for a specific column, null value counts, and partition-specific data, is used by the engine for file pruning.
        - Data layer
            - Query engines filter through the metadata files to read the datafiles required by a particular query efficiently.
            - On the write side, datafiles get written on the file storage, and the related metadata files are created and updated accordingly.
    - Writing Queries in Apache Iceberg
        - When a write query is initiated, it is sent to the engine for parsing. 
        - The catalog is then consulted to ensure consistency and integrity in the data and to write the data as per the defined partition strategies. 
        - The datafiles and metadata files are then written based on the query. 
        - Finally, the catalog file is updated to reflect the latest metadata, enabling subsequent read operations
    - Reading Queries in Apache Iceberg
        - When a read query is initiated, it is sent to the query engine first. 
        - The engine leverages the catalog to retrieve the latest metadata file location, which contains critical information about the tables schema and other metadata files, such as the manifest list that ultimately leads to the actual datafiles. 
        - Statistical information about columns is used in this process to limit the number of files being read, which helps improve query performance.

## Optimizing the Performance of Iceberg Tables
    - Compaction
        - in the world of streaming or “real-time” data, where data is ingested as it is created, generating lots of files with only a few records in each.
        - The solution to this problem is to periodically take the data in all these small files and rewrite it into fewer larger files
    - Compaction Strategies
        Binpack 
            - What it does - Combines files only; no global sorting (will do local sorting within tasks)
            - Pros - This offers the fastest compaction jobs.
            - Cons - Data is not clustered.
        Sort
            - What it does - Sorts by one or more fields sequentially prior to allocating tasks (e.g., sort by field a, then within that, sort by field b)
            - Pros - Data clustered by often queried fields can lead to much faster read times.
            - Cons - This results in longer compaction jobs compared to binpack.
        z-order 
            - What it does - Sorts by multiple fields that are equally weighted, prior to allocating tasks (X and Y values in this range are in one grouping; those in another range are in
            another grouping)
            - Pros - If queries often rely on filters on multiple fields, this can improve read times even further.
            - Cons - This results in longer running compaction jobs compared to binpack.
    - Keep in mind that compaction always honors the current partition spec, so if data from an old partition spec is rewritten, it will have the new partitioning rules applied.
    - Partitioning
        - When a table is partitioned, instead of just sorting the order based on a field, it will write records with distinct values of the target field into their own datafiles.
    - Hidden Partitioning
        - Instead of tracking it by relying on how files are physically laid out, Iceberg tracks the range of partition values at the snapshot and manifest levels
    - Partition Evolution
        - the metadata tracks not only partition values but also historical partition schemes, allowing the partition schemes to evolve. 
    - Row-level update modes in Apache Iceberg
        - Copy-on-write: 
            - In this approach, if even a single row in a datafile is updated or deleted, that datafile is rewritten, and the new file takes its place in the new snapshot.
            - Fastest reads, Slowest updates/deletes
        - Merge-on-read 
            - Instead of rewriting an entire datafile, you capture in a delete file the records to be updated in the existing file, with the delete file tracking which records should be ignored.
            - Merge-on-read (position deletes): Fast reads, Fast updates/deletes, Use regular compaction to minimize read costs.
            - Merge-on-read (equality deletes): Slow reads,  Fastest updates/deletes, Use more frequent compaction to minimize read costs.
    - Write Distribution Mode
        - how the data is distributed among tasks
        - Write Distribution Mode options:
            - none : There is no special distribution. This is the fastest during write time and is ideal for presorted data.
            - hash : The data is hash-distributed by partition key.
            - range : The data is range-distributed by partition key or sort order.
    - Datafile Bloom Filters
        - A bloom filter is a way of knowing whether a value possibly exists in a dataset.
        - Bloom filters are handy because they can help us avoid unnecessary data scans.

## Iceberg Catalogs
    - Hadoop Catalog
        - Hadoop catalog is that it doesn’t require any external systems to run. All it requires is a filesystem, thereby lowering the barrier to getting started with Iceberg.
        - it is not recommended for production usage.
    - Hive Catalog
        - It maps a table's path to its current metadata file by using the location table property in the table's entry in the Hive Metastore. The value of this property is the absolute path in the filesystem where the table's current metadata file can be located.
        - it is compatible with a wide variety of engines and tools, it is cloud agnostic.
        - it requires running an additional service yourself. 
        - it doesn't provide support for multitable transactions.
    - AWS Glue Catalog
        - It utilizes the AWS Glue Data catalog as a centralized metadata repository to track Iceberg table metadata. 
        - It maps a table's path to its current metadata file as a table property called metadata_location in the table's entry in Glue. The value for this property is the absolute path to the metadata file in the filesystem.
        - it does not support multitable transactions.
    - Nessie Catalog
        - a table's path is mapped to its current metadata file using a table property called metadataLocation stored within the table's entry in Nessie. This property's value is the absolute path of the current metadata file for the table.
        - it introduces a Git-like experience to data lakes and enables the concept of “data as code”
        - it enables multitable and multistatement transactions like a data warehouse does. 
        - it's cloud agnostic
        - not all engines and tools support Nessie as a catalog.
    - REST Catalog
        - REST catalog is an interface rather than a specific implementation.
        - it requires fewer packages and dependencies compared to other catalogs, simplifying deployment and management.
        - REST catalog supports multitable transactions, offering flexibility for complex operations across multiple tables.
        - it's cloud agnostic
        - you have to run a process to handle and respond to the REST calls from engines and tools.
    - JDBC Catalog
        - The JDBC catalog maps a table's path to its current metadata file via a table property called metadata_location in the JDBC-compliant database, storing the location of the current metadata file for the table.
        - it doesn't support multitable transactions. 
        - it requires all your engines and tools to either package a JDBC driver with its deployment or be able to pull one in dynamically, increasing the dependencies on your deployment.
    - Catalog Migration
        - A nice thing about the migration being lightweight is that you can continue using your current catalog, register a set of tables in the new catalog, keep the entry in your current catalog, and do testing on the new catalog. 
        - In this situation that the new catalog's entry will be stale, with any changes made to the table using the current catalog. You shouldn't make any changes to the table using the new catalog, as all existing usage of the current catalog won't see those changes.