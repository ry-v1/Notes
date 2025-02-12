# Learning Spark

### Introduction to Apache Spark: A Unified Analytics Engine

- Apache Spark is a unified engine designed for large-scale distributed data processing, on premises in data centers or in the cloud.
- Spark provides in-memory storage for intermediate computations.

##### Apache Spark's Distributed Execution

- A Spark application consists of a driver program that is responsible for orchestrating parallel operations on the Spark cluster. The driver accesses the distributed components in the cluster—the Spark executors and cluster manager—through a SparkSession.

- Spark driver : it communicates with the cluster manager; it requests resources (CPU, memory, etc.) from the cluster manager for Spark’s executors (JVMs); and it transforms all the Spark operations into DAG computations, schedules them, and distributes their execution as tasks across the Spark executors. Once the resources are allocated, it communicates directly with the executors.

- SparkSession : Through this one conduit, you can create JVM runtime parameters, define DataFrames and Datasets, read from data sources, access catalog metadata, and issue Spark SQL queries. SparkSession provides a single unified entry point to all of Spark’s functionality.

- Cluster manager : The cluster manager is responsible for managing and allocating resources for the cluster of nodes on which your Spark application runs. Currently, Spark supports four cluster managers: the built-in standalone cluster manager, Apache Hadoop YARN, Apache Mesos, and Kubernetes.

- Spark executor : A Spark executor runs on each worker node in the cluster. The executors communicate with the driver program and are responsible for executing tasks on the workers. In most deployments modes, only a single executor runs per node.

- Deployment modes : An attractive feature of Spark is its support for myriad deployment modes, enabling Spark to run in different configurations and environments. Because the cluster manager is agnostic to where it runs (as long as it can manage Spark’s executors and fulfill resource requests), Spark can be deployed in some of the most popular environments—such as Apache Hadoop YARN and Kubernetes—and can operate in different modes.

#### Understanding Spark Application Concepts

- Application : A user program built on Spark using its APIs. It consists of a driver program and executors on the cluster.

- SparkSession : An object that provides a point of entry to interact with underlying Spark functionality and allows programming Spark with its APIs. In an interactive Spark shell, the Spark driver instantiates a SparkSession for you, while in a Spark application, you create a SparkSession object yourself.

- Job : A parallel computation consisting of multiple tasks that gets spawned in response to a Spark action (e.g., save(), collect()).

- Stage : Each job gets divided into smaller sets of tasks called stages that depend on each other.

- Task : A single unit of work or execution that will be sent to a Spark executor.

- Catalyst optimizer takes a computational query and converts it into an execution plan.

    - Phase 1: Analysis : The Spark SQL engine begins by generating an abstract syntax tree (AST) for the SQL or DataFrame query. In this initial phase, any columns or table names will be resolved by consulting an internal Catalog, a programmatic interface to Spark SQL that holds a list of names of columns, data types, functions, tables, databases, etc. Once they've all been successfully resolved, the query proceeds to the next phase.
    - Phase 2: Logical optimization: This phase comprises two internal stages. Applying a standard rule based optimization approach, the Catalyst optimizer will first construct a set of multiple plans and then, using its cost-based optimizer (CBO), assign costs to each plan. These plans are laid out as operator trees; they may include, for example, the process of constant folding, predicate pushdown, projection pruning, Boolean expression simplification, etc. This logical plan is the input into the physical plan.
    - Phase 3: Physical planning : In this phase, Spark SQL generates an optimal physical plan for the selected logical plan, using physical operators that match those available in the Spark execution engine.
    - Phase 4: Code generation :  The final phase of query optimization involves generating efficient Java bytecode to run on each machine. Because Spark SQL can operate on data sets loaded in memory, Spark can use state-of-the-art compiler technology for code generation to speed up execution. In other words, it acts as a compiler. Project Tungsten, which facilitates whole-stage code generation, plays a role here.
    - what is whole-stage code generation? It's a physical query optimization phase that collapses the whole query into a single function, getting rid of virtual function calls and employing CPU registers for intermediate data. The second-generation Tungsten engine, introduced in Spark 2.0, uses this approach to generate compact RDD code for final execution. This streamlined strategy significantly improves CPU efficiency and performance.

- Managed Versus UnmanagedTables
    - Spark allows you to create two types of tables: managed and unmanaged. 
    - For a managed table, Spark manages both the metadata and the data in the file store. This could be a local filesystem, HDFS, or an object store such as Amazon S3 or Azure Blob. 
    - For an unmanaged table, Spark only manages the metadata, while you manage the data yourself in an external data source such as Cassandra.

- Temporary views versus global temporary views
    - A temporary view is tied to a single SparkSession within a Spark application. 
    - A global temporary view is visible across multiple SparkSessions within a Spark application.

#### Optimizing and Tuning Spark for Efficiency

- Viewing and Setting Apache Spark Configurations
    - In $SPARK_HOME directory (where you installed Spark), there are a number of config files: conf/spark-defaults.conf.template, conf/log4j.properties.template, and conf/spark-env.sh.template. Changing the default values in these files and saving them without the .template suffix instructs Spark to use these new values.
    - Set Spark configurations directly in your Spark application or on the command line when submitting the application with spark-submit, using the --conf flag
    - The third option is through a programmatic interface via the Spark shell.

- Static versus dynamic resource allocation
    - When you specify compute resources as command-line arguments to spark-submit, you cap the limit. This means that if more resources are needed later as tasks queue up in the driver due to a larger than anticipated workload, Spark cannot accommodate or allocate extra resources.
    - If instead you use Spark’s dynamic resource allocation configuration, the Spark driver can request more or fewer compute resources as the demand of large workloads flows and ebbs. In scenarios where your workloads are dynamic—that is, they vary in their demand for compute capacity—using dynamic allocation helps to accommodate sudden peaks.

- Configuring Spark executors’ memory and the shuffle service
    - The amount of memory available to each executor is controlled by spark.executor.memory. 
    - This is divided into three sections, as depicted in execution memory, storage memory, and reserved memory. 
    - The default division is 60% for execution memory and 40% for storage, after allowing for 300 MB for reserved memory, to safeguard against OOM errors.
    - When storage memory is not being used, Spark can acquire it for use in execution memory for execution purposes, and vice versa.
    - Execution memory is used for Spark shuffles, joins, sorts, and aggregations. Since different queries may require different amounts of memory, the fraction (spark.memory.fraction is 0.6 by default) of the available memory to dedicate to this can be tricky to tune but it’s easy to adjust. 
    - By contrast, storage memory is primarily used for caching user data structures and partitions derived from DataFrames.
    - During map and shuffle operations, Spark writes to and reads from the local disk’s shuffle files, so there is heavy I/O activity. This can result in a bottleneck, because the default configurations are suboptimal for large-scale Spark jobs.

- Spark configurations to tweak for I/O during map and shuffle operations
    Configuration | Default value, recommendation, and description
    spark.driver.memory | Default is 1g (1 GB). This is the amount of memory allocated to the Spark driver to receive data from executors. This is often changed during     | spark-submit with --driver-memory. Only change this if you expect the driver to receive large amounts of data back from operations like collect(), or if you run out of driver memory.
    spark.shuffle.file.buffer | Default is 32 KB. Recommended is 1 MB. This allows Spark to do more buffering before writing final map results to disk.
    spark.file.transferTo | Default is true. Setting it to false will force Spark to use the file buffer to transfer files before finally writing to disk; this will decrease the I/O activity.
    spark.shuffle.unsafe.file.output.buffer | Default is 32 KB. This controls the amount of buffering possible when merging files during shuffle operations. In general, large values (e.g., 1 MB) are more appropriate for larger workloads, whereas the default can work for smaller workloads.
    spark.io.compression.lz4.blockSize | Default is 32 KB. Increase to 512 KB. You can decrease the size of the shuffle file by increasing the compressed size of the block.
    spark.shuffle.service.index.cache.size | Default is 100m. Cache entries are limited to the specified memory footprint in byte.
    spark.shuffle.registration.timeout | Default is 5000 ms. Increase to 120000 ms.
    spark.shuffle.registration.maxAttempts | Default is 3. Increase to 5 if needed.

- Spark will at best schedule a thread per task per core, and each task will process a distinct partition. To optimize resource utilization and maximize parallelism, the ideal is at least as many partitions as there are cores on the executor.

- If there are more partitions than there are cores on each executor, all the cores are kept busy. You can think of partitions as atomic units of parallelism: a single thread running on a single core can work on a single partition.

- How partitions are created?
    - Spark’s tasks process data as partitions read from disk into memory. Data on disk is laid out in chunks or contiguous file blocks, depending on the store. By default, file blocks on data stores range in size from 64 MB to 128 MB. For example, on HDFS and S3 the default size is 128 MB (this is configurable). A contiguous collection of these blocks constitutes a partition.
    - The size of a partition in Spark is dictated by spark.sql.files.maxPartitionBytes. The default is 128 MB. You can decrease the size, but that may result in what’s known as the “small file problem”—many small partition files, introducing an inordinate amount of disk I/O and performance degradation thanks to filesystem operations such as opening, closing, and listing directories, which on a distributed filesystem can be slow.