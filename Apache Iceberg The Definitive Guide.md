# Apache Iceberg The Definitive Guide

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