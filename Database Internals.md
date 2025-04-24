# Database Internals

### Storage Engines
    - The primary job of any database management system is reliably storing data and making it available for users. We use databases as a primary source of data, helping us to share it between the different parts of our applications. Instead of finding a way to store and retrieve information and inventing a new way to organize data every time we create a new app, we use databases.
    - Databases are modular systems and consist of multiple parts: a transport layer accepting requests, a query processor determining the most efficient way to run queries, an execution engine carrying out the operations, and a storage engine.
    - The storage engine (or database engine) is a software component of a database management system responsible for storing, retrieving, and managing data in memory and on disk, designed to capture a persistent, long-term memory of each node.
    
##### DBMS Architecture
    - Database management systems use a client/server model, where database system instances (nodes) take the role of servers, and application instances take the role of clients.
    - Client requests arrive through the transport subsystem. Requests come in the form of queries, most often expressed in some query language. The transport subsystem is also responsible for communication with other nodes in the database cluster.
    - Upon receipt, the transport subsystem hands the query over to a query processor, which parses, interprets, and validates it. Later, access control checks are performed, as they can be done fully only after the query is interpreted.
    - The parsed query is passed to the query optimizer, which first eliminates impossible and redundant parts of the query, and then attempts to find the most efficient way to execute it based on internal statistics (index cardinality, approximate intersection size, etc.) and data placement (which nodes in the cluster hold the data and the costs associated with its transfer). The optimizer handles both relational operations required for query resolution, usually presented as a dependency tree, and optimizations, such as index ordering, cardinality estimation, and choosing access methods. 
    - The query is usually presented in the form of an execution plan (or query plan): a sequence of operations that have to be carried out for its results to be considered complete. Since the same query can be satisfied using different execution plans that can vary in efficiency, the optimizer picks the best available plan. The execution plan is handled by the execution engine, which collects the results of the execution of local and remote operations. Remote execution can involve writing and reading data to and from other nodes in the cluster, and replication.
    - Local queries (coming directly from clients or from other nodes) are executed by the storage engine. The storage engine has several components with dedicated responsibilities:
        - Transaction manager: This manager schedules transactions and ensures they cannot leave the database in a logically inconsistent state.
        - Lock manager: This manager locks on the database objects for the running transactions, ensuring that concurrent operations do not violate physical data integrity.
        - Access methods (storage structures): These manage access and organizing data on disk. Access methods include heap files and storage structures such as B-Trees or LSM Trees.
        - Buffer manager: This manager caches data pages in memory.
        - Recovery manager: This manager maintains the operation log and restoring the system state in case of a failure.
    - Together, transaction and lock managers are responsible for concurrency control: they guarantee logical and physical data integrity while ensuring that concurrent operations are executed as efficiently as possible.
    