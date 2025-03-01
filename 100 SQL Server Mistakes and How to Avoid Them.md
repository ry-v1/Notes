This file has notes from the book "100 SQL Server Mistakes and How to Avoid Them"

**C4 model**

- C4 model - The C4 model is a set of architectural standard diagrams.
  - At its core, C4 contains four standard diagrams:
    - System Context diagram
    - Container diagram
    - Component diagram
    - Code diagram
    
    The system context diagram is at the highest level, illustrating an application’s interfaces with users and other applications.
    Each subsequent level drills through a specific area of the application, providing ever more granular detail.
    The lowest level of granularity is the code diagram at the bottom.

  - The model also contains side diagrams:
    -  System landscape diagram, which illustrates how a portfolio of applications interacts
    -  Dynamic diagram, which illustrates how static elements work together at run time to form a feature
    -  Deployment diagram and illustrates the platform that an application is deployed on.

  - A full description can be found at https://c4model.com.

**Development standards**

  - Naming standards
    - Always use object names that are meaningful, with a view to making your code self-documenting.
    - Always consider database architecture, which is the design of your database, even within agile projects.
    - Avoid using prefixes for database objects, as they can actually make objects harder to locate.
    - Be especially careful to avoid using the sp_ prefix for stored procedures as this indicates that they are a system stored procedure, instead of a user-defined procedure.
  - Coding standards
    - Always make time to ensure that your data-tier application has coding standards. These should form part of your architectural efforts and should consider both stylistic choices as well as technical standards.
    - Avoid using ordinal column numbers

**Data types**

  - Constraints are vital for ensuring data quality within a database. For example, a foreign key will ensure that a value exists in a different table before it is inserted or updated. A NOT NULL constraint enforces that a value in the column is mandatory and that the column cannot contain NULL values.
  - Data type is also a constraint—a constraint that applies to every single column in every single table within every single database. It is the foundation of data quality and functionality.
  - Always use the most restrictive data type that will allow you to store all potentially required values. This is especially true for integer values that are used in key constraints or values that are indexed.
  - SMALLINT and TINYINT should be used instead of INT where we do not expect values to overflow these sizes, to reduce wasted space.
  - Always use fixed-length strings to reduce storage overhead when the length of strings is consistent within a column.
  - Always use variable-length strings when the length will not be consistent.

**Database design**

  - Relational databases join tables together using the concept of keys. There are two basic types of keys—a primary key and a foreign key.
    - A primary key is used to uniquely identify a row in a table.
    - A foreign key is used to point to a primary key value or a unique constraint from a secondary table.
    - If a key has a business meaning, such as SocialSecurityNumber, then it is known as a *natural key*.
    - If a key does not have business meaning, such as an arbitrary, incrementing number, like EmployeeID, then it is called an *artificial key*.
    - A natural key may consist of multiple columns. If a key consists of multiple columns, it is called a *composite key*. It’s best to minimize the use of composite keys when possible.
  - Avoid using wide columns or multiple columns as a primary key, as the primary key will often become the clustered index and thus will be replicated in your nonclustered indexes and can lead to performance degradation.
  - Always use a foreign key constraint, where tables are related to each other, to avoid data consistency issues.

**T-SQL development**

  - NULL is not equal to NULL.
  - Avoid SELECT * in anything other than ad hoc queries, as it can cause issues with performance and code maintenance, and it’s an anti-pattern for self-documenting code.
  - Only order data if it is absolutely necessary.
  - If DISTINCT causes performance issues, consider other techniques such as  GROUP BY or ROW_NUMBER().
  - Use UNION ALL instead of UNION if duplicates are either unimportant or not possible.
  - Cursors should be avoided. They are very expensive, and in modern versions of SQL Server, there are no operations that can’t be performed via other methods.
  - Deleting many rows from a table in a single transaction can cause the transaction log to become full. Avoid this by splitting the deletion into multiple batches.

**Error handling, testing, source control, and deployment**

  - ACID is a set of basic rules for transactions, which state that they must be atomic, consistent, isolated, and durable.
  - XACT_ABORT is used to determine if an entire transaction is terminated when low severity errors cause a statement to fail.
  - We should always write error handling for our code. Use TRY..CATCH to trap errors.
  - Use THROW and RAISERROR() to raise meaningful error messages.
  - Error severity levels, ranging from 0..24, denote the severity of an error, from informational message through to critical hardware or software failures.
  - If an application has unattended processes, consider alerting if errors are raised so that the application support team can deal with them.
  - Always keep T-SQL code in a source control repository. This will provide a version history and a rollback mechanism, as well as help streamline the development of a project with multiple developers.
  - Always write unit tests for programmable objects. Doing so will take a little more time up front but will save more time when resolving issues with bugs breaking existing functionality.
  - Use modern deployment techniques. A CI/CD pipeline can provide many benefits, including improving time to market and reducing deployment complexity.

**Optimization**
  - Instance file initialization should be used in the vast majority of situations. It should only be avoided in the most secure of environments.
  - Lock Pages In Memory should be used in many situations. There are times when it should be disabled in private clouds, but public cloud providers recommend it.
  - Always leave enough RAM for the operating system and any other applications that are running on the server. This is especially important if Lock Pages In Memory is used.
  - Use the Max Server Memory setting to configure the maximum amount of RAM that can be allocated to the SQL Server buffer pool.
  - Always try to work with the optimizer, rather than against it. In rare situations when query hints are required, try to leave the optimizer with multiple options.
  - Remember to take advantage of DOP query feedback.
  - DOP feedback is disabled by default, unlike other query feedback mechanisms.
  - Consider partitioning large tables to improve performance and reduce load on the I/O subsystem.
  - Partitioning tables allows for partition elimination, meaning that partitions are not read if they do not store relevant data.
  - Consider using data compression as a performance enhancement for large tables.
  - Data compression is best suited to workloads that are I/O bound and have spare processor capacity.
  - The compression rate achieved will be dependent on the data stored within the table. Use the sp_estimate_data_compression_savings stored procedure to evaluate the impact of row compression and page compression before implementing either option.
  - Avoid the Read Uncommitted transaction isolation level unless you are working with tables stored in read-only filegroups.
  - The Read Uncommitted isolation level can result in dirty reads just as the use of the NOLOCK query hint can.
  - Avoid using strong isolation levels such as Repeatable Read or Serializable unless they are absolutely required, as they can lead to lock contention and deadlocks.
  - Consider using optimistic isolation levels in situations when you face performance issues caused by lock contention.
  - The optimistic isolation levels are Read Committed Snapshot and Snapshot.
  - Optimistic isolation levels are best suited to environments where TempDB is not I/O bound and there is plenty of free space on the TempDB volume.
  - Avoid throwing extra hardware at performance issues. Try to diagnose the root cause of the issue and resolve it instead.

**Indexes**
  - If we create a table with no clustered index, this is known as a heap. In a heap, data is stored in no particular order. 
  - Instead of an index root, there is a simple Index Allocation Map that stores a list of all pages allocated to the heap. 
  - For larger tables, this means that SQL Server has to work hard to find any given value. 
  - Specifically, almost any query SELECT statement issued against a heap will require SQL Server to read every single data page that makes up the table. 
  - Queries that include keywords such as TOP may not require a read of all pages in the table.

  - If we create a clustered index on a table, then SQL Server builds a B-tree structure. 
  - This operation orders the pages in a table using the clustered key. 
  - The clustered key is usually built on the primary key of the table. 
  - If you have a wide primary key, however, it is possible to build it on a different, unique column.
  - A clustered index allows for read operations to be performed more efficiently. 
  - The leaf level of the B-tree structure is the actual data pages of the table. 
  - This means that the data pages of the table are stored in the order of the clustered key. For this reason, we can only have a single clustered index on a table.
  - A B-tree structure is the structure that indexes are organized into. They have a root level consisting of a single page, zero or more intermediate levels, and a singl leaf level.
  - The leaf level of a clustered index is the actual data pages of the table whereas the leaf level of a nonclustered index contains pointers to the data pages of the table.

  - A nonclustered index is a B-tree structure built on a different column(s) within the table. 
  - SQL Server can use these indexes to improve the performance of operations such as joins, filters, and aggregations. 
  - The leaf level of a nonclustered index contains pointers to the data pages of the heap or clustered index. 
  - Because it does not impact the order of the actual data pages, we can have multiple nonclustered indexes on a table. 
  - In fact, a table can support up to 256 nonclustered indexes, although having too many can have a negative impact on write operations and also consumes space on disk, and potentially in memory.

  - Internal fragmentation describes the amount of free space on index pages.
  - Internal fragmentation refers to a low page density, which causes more pages than necessary to be read.
  - Low page density needs to be traded off against the risk of page splits caused by updates to very dense pages.

  - External fragmentation refers to index pages becoming out of physical order. 
  - External fragmentation refers to pages being out of order, which can damage performance.
  - External fragmentation only causes a performance issue for index scans. Index seeks are not impacted.

  - Good page splits occur when pages are allocated at the end of an index.
  - Bad page splits occur when pages are allocated in the middle of the index and data needs to be moved to the new page. These page splits cause increased I/O and performance penalties.

  - An index seek starts at the root level of the B-tree and traverses all levels until it finds the required row.
  - An index scan reads the leaf level of an index until it reaches the end of the data it is searching for.
  - An index loop uses a nonclustered index to perform a filter or aggregation and then looks up further data from the clustered index or heap.
  - Seek, scan, and lookup ratios can be determined by using the sys.dm_db_index_usage_stats dynamic management view (DMV).
  - Do not reorganize indexes to fix page density. Reorganizing indexes only fills pages up to the level of FILLFACTOR. It does not reduce density to the level of FILLFACTOR. We should rebuild indexes instead.
  - Avoid aggregating fragmentation statistics from sys.dm_db_index_physical_stats. Instead, focus on the leaf-level data.
  - Not rebuilding indexes will have an impact on index scans and therefore query performance. If a database is 24/7, use online index rebuilds.
  - Do not rebuild all indexes indiscriminately. Only rebuild indexes that require it based on fragmentation statistics.
  - Do not update statistics after rebuilding indexes, as this can result in worse statistics.
  - In the majority of cases, automatically updating statistics is good enough.
  - If you decide to update statistics, consider the tradeoff against plan recompilation.
  - Depending on the query performance tradeoff against a maintenance window, consider MAXDOP for index rebuilds.
  - If you are suffering from last page insert contention, consider using OPTIMIZE_FOR_SEQUENTIAL_KEY.
  - If you need to perform bulk load operations, consider disabling nonclustered indexes on the target table to improve write performance.
  - Do not rely too heavily on tools such as Database Engine Tuning Advisor. You can use them as a guide, but you must layer them with your own business knowledge.
  - Columnstore indexes organize pages around columns instead of rows.
  - Consider using columnstore indexes on large fact tables to improve the performance of analytical queries.