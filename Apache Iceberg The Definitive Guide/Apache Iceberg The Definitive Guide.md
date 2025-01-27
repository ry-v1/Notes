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

## Apache Iceberg in Production
    - history Metadata Table
        - made_current_at : represents the exact timestamp when the corresponding snapshot was made the current snapshot. This gives you a precise temporalmarker for when changes to the table were committed.
        - snapshot_id : serves as a unique identifier for each snapshot. This identifier enables you to track and reference specific snapshots within the table's history.
        - parent_id : provides the unique ID of the parent snapshot of the current snapshot. This effectively maps out the lineage of each snapshot, thus facilitating the tracking of the table's evolution over time.
        - is_current_ancestor : indicates whether a snapshot is an ancestor of the table's current snapshot. This boolean value (true or false) helps identify snapshots that are part of the table's present state lineage and those that have been invalidated from table rollbacks.

    - metadata_log_entries Metadata Table
        - timestamp : records the exact date and time when the metadata was updated. This timestamp serves as a temporal marker for the state of the table at that specific moment.
        - file : indicates the location of the datafile that corresponds to that particular metadata log entry. This location acts as a reference point to access the actual data associated with the metadata entry.
        - latest_snapshot_id : provides the identifier of the most recent snapshot at the time of the metadata update. It is a useful reference point for understanding the state of the data when the metadata was updated.
        - latest_schema_id : contains the ID of the schema being used when the metadata log entry was created. This gives context about the structure of the data at the time of the metadata update.
        - latest_sequence_number field signifies the order of the metadata updates. It's an incrementing count that helps track the sequence of metadata changes over time.
    
    - snapshots Metadata Table
        - committed_at : signifies the precise timestamp when the snapshot was created, giving an indication of when the snapshot and its associated data state were committed.
        - snapshot_id : is a unique identifier for each snapshot. This field is crucial for distinguishing between the different snapshots and for specific operations such as snapshot retrieval or deletion.
        - operation : lists a string of the types of operations that occurred, such as APPEND and OVERWRITE.
        - parent_id : links to the snapshot ID of the snapshot's parent, providing context about the lineage of snapshots and allowing for the reconstruction of a historical sequence of snapshots.
        - manifest_list : offers detailed insights into the files comprising the snapshot. It's like a directory or inventory that keeps a record of all the datafiles associated with a given snapshot.
        - summary : holds metrics about the snapshot, such as the number of added or deleted files, number of records, and other statistical data that provides a quick glance into the snapshot's content.
    
    - files Metadata Table : showcases the current datafiles within a table and furnishes detailed information about each of them, from their location and format to their content and partitioning specifics.
        - content : represents the type of content in the file, with a 0 signifying a datafile, 1 a position delete file, and 2 an equality delete file.
        - file_path : gives the exact location of each file. This helps facilitate access to each datafile when needed.
        - file_format : indicates the format of the datafile; for instance, whether it's a Parquet, Avro, or ORC file.
        - spec_id : corresponds to the partition spec ID that the file adheres to, providing a reference to how the data is partitioned.
        - partition : provides a representation of the datafile's specific partition, indicating how the data within the file is divided for optimized access and query performance.
        - record_count : reports the number of records contained within each file, giving a measure of the file's data volume.
        - file_size_in_bytes : provides the total size of the file in bytes, while column_sizes furnishes the sizes of the individual columns.
        - value_counts, null_value_counts, and nan_value_counts : provide the count of non-null, null, and NaN (Not a Number) values, respectively, in each column.
        - lower_bounds and upper_bounds : hold the minimum and maximum values in each column, providing essential insights into the data range within each file.
        - key_metadata : contains implementation-specific metadata, if any exists.
        - split_offsets : provides the offsets at which the file is split into smaller segments for parallel processing.
        - equality_ids and sort_order_id : correspond to the IDs relating to equality delete files, if any exist, and the IDs of the table's sort order, if it has one.
    
    - manifests Metadata Table
        - path : provides the filepath where the manifest is stored, enabling quick access to the file. 
        - length : on the other hand, shows the size of the manifest file.
        - partition_spec_id : indicates the specification ID of the partition that the manifest file is associated with, which is valuable for tracking changes in partitioned tables. 
        - added_snapshot_id : provides the ID of the snapshot that added this manifest file, offering a link between snapshots and manifests.
        - added_data_files_count, existing_data_files_count, and deleted_data_files_count : respectively relay the number of new files added in this manifest, the number of existing datafiles that were added in previous snapshots, and the number of files deleted in this manifest. 
        - partition_summaries : is an array of field_summary structs that summarize partition-level statistics. It contains the following information: contains_null, contains_nan, lower_bound, and upper_bound. These fields indicate whether the partition contains null or NaN values, and they provide the lower and upper bounds of data within the partition.
    
    - partitions Metadata Table
        - partition : represents the actual partition values, usually based on certain columns of your data. This allows your data to be organized in a meaningful way and enables efficient query processing as data can be retrieved based on specific partition values.
        - record_count : indicates the total number of records within a given partition. This metric can be helpful in understanding data distribution across the partitions and can guide optimization strategies such as repartitioning and rebalancing.
        - file_count : gives the total number of datafiles present in the partition. It's crucial in managing and optimizing storage, as having too many small files can impact query performance.
        - spec_id : corresponds to the ID of the partition specification used to generate this partition. Partition specifications define how the data is split into partitions, and having the ID readily available aids in understanding the partitioning strategy used.
        - position_delete_record_count, position_delete_file_count, equality_delete_record_count, and equality_delete_file_count

    - all_data_files Metadata Table : provides comprehensive details about every datafile across all valid snapshots in the table.
        - content : signifies the type of the file. A value of 0 indicates a datafile, 1 a position delete file, and 2 an equality delete file.
        - file_path : is a string that represents the complete path to the datafile. This usually includes the storage system location (e.g., s3://my-bucket/folder/subfolder/myfile.xyz), the table name, and the unique file identifier.
        - file_format : indicates the format of the datafile. In our example, it's Parquet, but it could be another file format such as AVRO or ORC.
        - spec_id : corresponds to the ID of the partition specification used to generate this partition.
        - partition : represents the partition to which this datafile belongs. It's usually based on the partitioning scheme defined for the table.
        - record_count : gives the total number of records within the file, while file_size_in_bytes represents the size of the datafile in bytes. Both metrics are essential for understanding the volume of data and can be used in query optimization strategies.
        - column_sizes : provides a map between the column ID and the size of that column in bytes.
        - value_counts : gives a map that represents the total count of values for each column in the datafile. 
        - null_value_counts and nan_value_counts provide a count of null and NaN values for each column.
        - lower_bounds and upper_bounds : are maps that store the minimum and maximum values for each column in the datafile. These fields are instrumental in pruning data during query execution.
        - key_metadata : contains implementation-specific metadata.
        - split_offsets : provides information about split points within the file. It's an array of long values and is especially useful in distributed processing scenarios, where datafiles can be split into smaller chunks for parallel processing.
        - equality_ids : relates to equality deletes and helps in identifying rows deleted by equality deletes.
        - sort_order_id : contains the ID of the sort order used to write the datafile.
        - readable_metrics : is a derived field that provides a human-readable representation of the file's metadata including column size, value counts, null counts, and lower and upper bounds.
       
       - all_data_files metadata table may produce more than one row per datafile because a file could be part of multiple table snapshots.
    
    - all_manifests Metadata Table : provides detailed insights into every manifest file across all valid snapshots in the table.
        - content : signifies the type of the file, similar to the all_data_files table. A value of 0 indicates the manifest tracks datafiles; a value of 1 indicates that it tracks delete files.
        - path : is a string representing the complete path to the manifest file. Like the all_data_files table, this includes the storage system location (e.g., s3://...), the table name, and a unique file identifier.
        - length : represents the size of the manifest file in bytes. This can provide insights into the volume of metadata stored in the manifest.
        - partition_spec_id : corresponds to the ID of the partition specification used to write this manifest file. This indicates how the datafiles listed in the manifest are partitioned.
        - added_snapshot_id : represents the ID of the snapshot when the manifest was created.
        - added_data_files_count, existing_data_files_count, and deleted_data_files_count : provide a summary of the changes in datafiles that this manifest file represents. 
        - added_delete_files_count, existing_delete_files_count, and deleted_delete_files_count : provide a similar summary for delete files.
        - partition_summaries : is an array of structures, where each structure provides a summary for a specific partition in the manifest file. Each structure indicates whether the partition contains null or NaN values, as well as the lower and upper bounds of the partition.
        - reference_snapshot_id : represents the ID of the snapshot that this record is associated with. You'll see a manifest listed once for each snapshot it was valid for.

    - refs Metadata Table : provides a list of all the named references within an Iceberg table. Named references can be thought of as pointers to specific snapshots of the table data, providing an ability to bookmark or version the table state.
        - name : represents the unique identifier for a named reference. Named references are categorized into two types, which brings us to the second field, type. The type can be one of two values: BRANCH, a mutable reference that can be moved to a new snapshot; or TAG, an immutable reference that, once created, always points to the same snapshot.
        - max_reference_age_in_ms : indicates the maximum duration in milliseconds that a snapshot can be referenced. This age is measured from the time the snapshot was added to the table. If the age of a snapshot exceeds this duration, it will no longer be valid and will be a candidate for cleanup during maintenance operations.
        - min_snapshots_to_keep : provides a lower limit on the number of snapshots to keep in the table history. The Iceberg table will always maintain at least this many snapshots, even if they are older than the max_snapshot_age_ms setting.
        - max_snapshot_age_in_ms : indicates the maximum age in milliseconds for any snapshot in the table. Snapshots that exceed this age could be removed by the maintenance operations, unless they are protected by the min_snapshots_to_keep setting.

    - entries Metadata Table : offers insightful details about each operation that has been performed on the table's data and deletes files across all snapshots. Each row in this table captures operations that affected many files at a certain point in the table's history, making it an essential resource for understanding the evolution of your dataset.
        - status : is an integer that indicates whether a file was added or deleted in the snapshot. A value of 0 represents an existing file, while 1 indicates an added file and 2 a deleted file. This field allows you to track the lifecycle of each file, providing a glimpse into the changes and modifications the dataset has undergone over time.
        - snapshot_id : is the unique identifier of the snapshot in which the operation took place. This ID allows you to connect each file operation to a particular snapshot, which can be beneficial in tracking changes made in specific versions of the table.
        - sequence_number : indicates the order of operations. This is a global counter across all snapshots of the table, and it increments for each change made, whether the change is an addition, a modification, or a deletion. By understanding the sequence number, you can reconstruct the exact series of operations that led to the current state of the table.
        - data_file : is a struct that encapsulates extensive details about the file involved in the operation. The struct includes fields such as the following:
            - file_path : The complete path to the file in the storage system
            - file_format : The format of the file, such as Parquet or AVRO
            - partition : Information about the partition the file belongs to
            - record_count : The total number of records in the file
            - file_size_in_bytes : The size of the file in bytes
            - column_sizes : A map of the size in bytes of each column
            - value_counts : A map with a count of total values in each column
            - null_value_counts : A map with a count of null values in each column
            - nan_value_counts : A map with a count of NaN values in each column
            - lower_bounds and upper_bounds : Maps containing the minimum and maximum values of each column
            - key_metadata : Implementation-specific metadata
            - split_offsets : Information about split points within the file

    - Isolation of Changes with Branches
        - at the table level, which is native to Apache Iceberg regardless of catalog.
            - isolating changes at the table level, involves creating branches for specific tables. 
            - Each branch contains a full history of changes made to that table. 
            - This approach allows for concurrent schema evolution, rollbacks, and other advanced use cases. 
            - It is a powerful tool for handling table-specific changes, but it lacks the ability to provide a holistic view of changes across the entire data catalog.
        - at the catalog level, which is possible when using the Project Nessie catalog.
            - Isolating changes at the catalog level allows you to manage a complete data lake as a single entity, capturing changes across multiple tables within a branch. 
            - Using Nessie, you can take a snapshot of the entire catalog at a particular point in time.
            - This practice facilitates a more comprehensive version control strategy, enabling you to test data transformations, track data lineage, and maintain data integrity across multiple tables.

        - Table-level isolation provides granular control and flexibility for individual tables but might become complex to manage in a large-scale data environment. 
        - Catalog-level isolation provides a comprehensive, unified view of changes but might be overkill for small-scale or single-table scenarios.

    - Table Branching  
        - ability for the metadata to track snapshots under different paths, known as branching

    - Table Tagging
        - ability to give particular snapshots a name, known as tagging.

    - Catalog branching
        - By creating a new branch, you can safely ingest and validate batches of data across multiple tables, reducing the risk of erroneous entries across your catalog of tables.
    
    - Catalog tagging
        - allows you to mark specific versions of your data, providing an easy way to track and reproduce states of the data at different points in time.
    
    - Multitable Transactions
        - In a multitable transaction, multiple operations, possibly involving different tables, are treated as a single atomic unit of work. 
        - This means that either all operations in the transaction succeed, or if any operation fails, all the changes made within that transaction are rolled back, thereby leaving the database in a consistent state.

    - Rolling Back at the Table Level
        - Table-level rollbacks are a specific data management technique that allows changes to a table to be reversed or “rolled back” to a previous state.
        - rollback_to_snapshot
        - rollback_to_timestamp
        - set_current_snapshot
        - cherrypick_snapshot

    - Rolling Back at the Catalog Level
        - Nessie as Apache Iceberg catalog has the ability to roll back data at the catalog level.

## Streaming with Apache Iceberg

    - Streaming with Spark
        - Fault tolerance
            - Spark Streaming is designed to be resilient to failures, with built-in recovery mechanisms. If a node goes down during a computation, the system can recover quickly and continue processing.
        - Integration
            - Spark Streaming integrates seamlessly with other Spark components such as Spark MLlib and Spark SQL, enabling powerful combined use cases. For example, you can use MLlib to build a machine learning (ML) model on a large dataset with Spark and then apply that model to a live data stream with Spark Streaming.
        - Real-time processing
            - Spark Streaming can process live data streams in real time. It divides incoming data into batches, which are then processed by the Spark engine to generate the final stream of results in batches.
        - Window operations 
            - Spark Streaming provides windowed computations, where transformations on Resilient Distributed Datasets (RDDs), Spark's fundamental data structure, can be applied over a sliding window of data. This is useful in many scenarios, such as computing trends over the last few hours in a data stream.
        - High throughput
            - Spark Streaming is designed to process a large amount of data, making it suitable for applications that need to process high-volume live data streams.
        - Multiple data sources
            - Spark Streaming can ingest data from various sources, including Kafka, Flume, and Kinesis, among others.

        - Apache Spark's microbatching approach sets it apart from other streaming engines, making it a compelling choice for real-time data processing. 
        - This approach involves processing data in small, discrete batches rather than handling each data point individually. 
        - This design decision represents a trade-off between factors such as latency, throughput, and cost.

        - Streaming into Iceberg with Spark
            - A key feature of this integration is the support for processing incremental data, which starts from a historical timestamp. 
            - Only append snapshots are supported in the context of streaming reads from an Iceberg table.
            - Iceberg supports two output modes: append, which appends rows of every microbatch to the table; and complete, which replaces the table contents at each microbatch.
        - Streaming from Iceberg with Spark
            - For streaming read operations from an Iceberg table, its essential to note that Iceberg only supports reading data from append snapshots.
            