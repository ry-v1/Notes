# Learning Spark

### Introduction to Apache Spark: A Unified Analytics Engine

- Apache Spark is a unified engine designed for large-scale distributed data processing, on premises in data centers or in the cloud.
- Spark provides in-memory storage for intermediate computations.

##### Apache Spark's Distributed Execution

- A Spark application consists of a driver program that is responsible for orchestrating parallel operations on the Spark cluster. The driver accesses the distributed components in the cluster—the Spark executors and cluster manager—through a SparkSession.

- Spark driver : it communicates with the cluster manager; it requests resources (CPU, memory, etc.) from the cluster manager for Spark's executors (JVMs); and it transforms all the Spark operations into DAG computations, schedules them, and distributes their execution as tasks across the Spark executors. Once the resources are allocated, it communicates directly with the executors.

- SparkSession : Through this one conduit, you can create JVM runtime parameters, define DataFrames and Datasets, read from data sources, access catalog metadata, and issue Spark SQL queries. SparkSession provides a single unified entry point to all of Spark's functionality.

- Cluster manager : The cluster manager is responsible for managing and allocating resources for the cluster of nodes on which your Spark application runs. Currently, Spark supports four cluster managers: the built-in standalone cluster manager, Apache Hadoop YARN, Apache Mesos, and Kubernetes.

- Spark executor : A Spark executor runs on each worker node in the cluster. The executors communicate with the driver program and are responsible for executing tasks on the workers. In most deployments modes, only a single executor runs per node.

- Deployment modes : An attractive feature of Spark is its support for myriad deployment modes, enabling Spark to run in different configurations and environments. Because the cluster manager is agnostic to where it runs (as long as it can manage Spark's executors and fulfill resource requests), Spark can be deployed in some of the most popular environments—such as Apache Hadoop YARN and Kubernetes—and can operate in different modes.

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
    - If instead you use Spark's dynamic resource allocation configuration, the Spark driver can request more or fewer compute resources as the demand of large workloads flows and ebbs. In scenarios where your workloads are dynamic—that is, they vary in their demand for compute capacity—using dynamic allocation helps to accommodate sudden peaks.

- Configuring Spark executors memory and the shuffle service
    - The amount of memory available to each executor is controlled by spark.executor.memory. 
    - This is divided into three sections, as depicted in execution memory, storage memory, and reserved memory. 
    - The default division is 60% for execution memory and 40% for storage, after allowing for 300 MB for reserved memory, to safeguard against OOM errors.
    - When storage memory is not being used, Spark can acquire it for use in execution memory for execution purposes, and vice versa.
    - Execution memory is used for Spark shuffles, joins, sorts, and aggregations. Since different queries may require different amounts of memory, the fraction (spark.memory.fraction is 0.6 by default) of the available memory to dedicate to this can be tricky to tune but it’s easy to adjust. 
    - By contrast, storage memory is primarily used for caching user data structures and partitions derived from DataFrames.
    - During map and shuffle operations, Spark writes to and reads from the local disk's shuffle files, so there is heavy I/O activity. This can result in a bottleneck, because the default configurations are suboptimal for large-scale Spark jobs.

- Spark configurations to tweak for I/O during map and shuffle operations
    
    | Configuration | Default value, recommendation, and description     | 
    | ------------- | -------------------------------------------------- | 
    | spark.driver.memory | Default is 1g (1 GB). This is the amount of memory allocated to the Spark driver to receive data from executors. This is often changed during spark-submit with --driver-memory. Only change this if you expect the driver to receive large amounts of data back from operations like collect(), or if you run out of driver memory.    | 
    | spark.shuffle.file.buffer | Default is 32 KB. Recommended is 1 MB. This allows Spark to do more buffering before writing final map results to disk.    | 
    | spark.file.transferTo | Default is true. Setting it to false will force Spark to use the file buffer to transfer files before finally writing to disk; this will decrease the I/O activity.    | 
    | spark.shuffle.unsafe.file.output.buffer | Default is 32 KB. This controls the amount of buffering possible when merging files during shuffle operations. In general, large values (e.g., 1 MB) are more appropriate for larger workloads, whereas the default can work for smaller workloads.    | 
    | spark.io.compression.lz4.blockSize | Default is 32 KB. Increase to 512 KB. You can decrease the size of the shuffle file by increasing the compressed size of the block.    | 
    | spark.shuffle.service.index.cache.size | Default is 100m. Cache entries are limited to the specified memory footprint in byte.    | 
    | spark.shuffle.registration.timeout | Default is 5000 ms. Increase to 120000 ms.    | 
    | spark.shuffle.registration.maxAttempts | Default is 3. Increase to 5 if needed.    | 

- Spark will at best schedule a thread per task per core, and each task will process a distinct partition. To optimize resource utilization and maximize parallelism, the ideal is at least as many partitions as there are cores on the executor.

- If there are more partitions than there are cores on each executor, all the cores are kept busy. You can think of partitions as atomic units of parallelism: a single thread running on a single core can work on a single partition.

- How partitions are created?
    - Spark's tasks process data as partitions read from disk into memory. Data on disk is laid out in chunks or contiguous file blocks, depending on the store. By default, file blocks on data stores range in size from 64 MB to 128 MB. For example, on HDFS and S3 the default size is 128 MB (this is configurable). A contiguous collection of these blocks constitutes a partition.
    - The size of a partition in Spark is dictated by spark.sql.files.maxPartitionBytes. The default is 128 MB. You can decrease the size, but that may result in what's known as the “small file problem”—many small partition files, introducing an inordinate amount of disk I/O and performance degradation thanks to filesystem operations such as opening, closing, and listing directories, which on a distributed filesystem can be slow.

- Caching and Persistence of Data
    - cache() will store as many of the partitions read in memory across Spark executors as memory allows
    - persist(StorageLevel.LEVEL) is nuanced, providing control over how your data is cached via StorageLevel.
    - When you use cache() or persist(), the DataFrame is not fully cached until you invoke an action that goes through every record(e.g., count()). 
    - If you use an action like take(1), only one partition will be cached because Catalyst realizes that you do not need to compute all the partitions just to retrieve one record.

- When to Cache and Persist
    - Common use cases for caching are scenarios where you will want to access a large data set repeatedly for queries or transformations. Some examples include:
        - DataFrames commonly used during iterative machine learning training
        - DataFrames accessed commonly for doing frequent transformations during ETL or building data pipelines

- When Not to Cache and Persist
    - Not all use cases dictate the need to cache. Some scenarios that may not warrant caching your DataFrames include:
        - DataFrames that are too big to fit in memory
        - An inexpensive transformation on a DataFrame not requiring frequent use, regardless of size

- Spark join strategies : by which it exchanges, moves, sorts, groups, and merges data across executors

    - Broadcast Hash Join : Also known as a map-side-only join
        - The broadcast hash join is employed when two data sets, one small (fitting in the driver's and executor's memory) and another large enough to ideally be spared from movement, need to be joined over certain conditions or columns. 
        - Using a Spark broadcast variable, the smaller data set is broadcasted by the driver to all Spark executors, and subsequently joined with the larger data set on each executor. 
        - This strategy avoids the large exchange.
        - By default Spark will use a broadcast join if the smaller data set is less than 10 MB.
        - This configuration is set in spark.sql.autoBroadcastJoinThreshold. Specifying a value of -1 in spark.sql.autoBroadcastJoinThreshold will cause Spark to always resort to a shuffle sort merge join
    - When to use a broadcast hash join : Use this type of join under the following conditions for maximum benefit:
        - When each key within the smaller and larger data sets is hashed to the same partition by Spark
        - When one data set is much smaller than the other (and within the default config of 10 MB, or more if you have sufficient memory)
        - When you only want to perform an equi-join, to combine two data sets based on matching unsorted keys
        - When you are not worried by excessive network bandwidth usage or OOM errors, because the smaller data set will be broadcast to all Spark executors
    
    - Shuffle Sort Merge Join
        - As the name indicates, this join scheme has two phases: a sort phase followed by a merge phase. 
        - The sort phase sorts each data set by its desired join key; the merge phase iterates over each key in the row from each data set and merges the rows if the two keys match.
        - By default, the SortMergeJoin is enabled via spark.sql.join.preferSortMergeJoin.
    - Optimizing the shuffle sort merge join
        - We can eliminate the Exchange step from this scheme if we create partitioned buckets for common sorted keys or columns on which we want to perform frequent equi-joins. 
        - We can create an explicit number of buckets to store specific sorted columns (one key per bucket). Presorting and reorganizing data in this way boosts performance, as it allows us to skip the expensive Exchange operation and go straight to WholeStageCodegen.
    - When to use a shuffle sort merge join : Use this type of join under the following conditions for maximum benefit:
        - When each key within two large data sets can be sorted and hashed to the same partition by Spark
        - When you want to perform only equi-joins to combine two data sets based on matching sorted keys
        - When you want to prevent Exchange and Sort operations to save large shuffles across the network

- Inspecting the Spark UI
    - Jobs and Stages tabs allow you to navigate through these and drill down to a granular level to examine the details of individual tasks. 

    - Jobs tab 
        - You can view their completion status and review metrics related to I/O, memory consumption, duration of execution, etc.
        - Jobs tab with the expanded Event Timeline, showing when executors were added to or removed from the cluster. It also provides a tabular list of all completed jobs in the cluster. 
        - The Duration column indicates the time it took for each job (identified by the Job Id in the first column) to finish. If this time is high, it's a good indication that you might want to investigate the stages in that job to see what tasks might be causing delays. 
        - From the summary page you can also access a details page for each job, including a DAG visualization and list of completed stages.
    - Stages tab
        - The Stages tab provides a summary of the current state of all stages of all jobs in the application. 
        - You can also access a details page for each stage, providing a DAG and metrics on its tasks.
        - You can see the average duration of each task, time spent in garbage collection (GC), and number of shuffle bytes/records read. 
        - If shuffle data is being read from remote executors, a high Shuffle Read Blocked Time can signal I/O issues. 
        - A high GC time signals too many objects on the heap (your executors may be memory-starved). 
        - If a stage's max task time is much larger than the median, then you probably have data skew caused by uneven data distribution in your partitions.
    - Executors
        - The Executors tab provides information on the executors created for the application.
        - We can drill down into the minutiae of details about resource usage (disk, memory, cores), time spent in GC, amount of data written and read during shuffle, etc.
        - In addition to the summary statistics, you can view how memory is used by each individual executor, and for what purpose. 
        - This also helps to examine resource usage when you have used the cache() or persist() method on a DataFrame or managed table.
    - Storage
        - The Storage tab, provides information on any tables or DataFrames cached by the application as a result of the cache() or persist() method.
    - SQL
        - The effects of Spark SQL queries that are executed as part of your Spark application are traceable and viewable through the SQL tab. 
        - You can see when the queries were executed and by which jobs, and their duration.
        - Clicking on the description of a query displays details of the execution plan with all the physical operators.
        - Under each physical operator of the plan—here, Scan In-memory table, HashAggregate, and Exchange—are SQL metrics.
        - These metrics are useful when we want to inspect the details of a physical operator and discover what transpired: how many rows were scanned, how many shuffle bytes were written, etc.
    - Environment
        - what environment variables are set, what jars are included, what Spark properties are set (and their respective values), what system properties are set, what runtime environment (such as JVM or Java version) is used

#### Structured Streaming
    - Five Steps to Define a Streaming Query
        - Step 1: Define input sources - spark.readStream to create a DataStreamReader
        - Step 2: Transform data
            - Stateless transformations : Operations like select(), filter(), map(), etc. do not require any information from previous rows to process the next row; each row can be processed by itself. The lack of previous “state” in these operations make them stateless. Stateless operations can be applied to both batch and streaming DataFrames.
            - Stateful transformations : In contrast, an aggregation operation like count() requires maintaining state to combine data across multiple rows. More specifically, any DataFrame operations involving grouping, joining, or aggregating are stateful transformations. While many of these operations are supported in Structured Streaming, a few combinations of them are not supported because it is either computationally hard or infeasible to compute them in an incremental manner.
        - Step 3: Define output sink and output mode - define how to write the processed output data with DataFrame.writeStream
        - Step 4: Specify processing details
            - Triggering details This indicates when to trigger the discovery and processing of newly available streaming data. There are four options:
                - Default
                    - When the trigger is not explicitly specified, then by default, the streaming query executes data in micro-batches where the next micro-batch is triggered as soon as the previous micro-batch has completed.
                - Processing time with trigger interval
                    - You can explicitly specify the ProcessingTime trigger with an interval, and the query will trigger micro-batches at that fixed interval.
                - Once
                    - In this mode, the streaming query will execute exactly one micro-batch—it processes all the new data available in a single batch and then stops itself. This is useful when you want to control the triggering and processing from an external scheduler that will restart the query using any custom schedule (e.g., to control cost by only executing a query once per day).
                - Continuous
                    - This is an experimental mode (as of Spark 3.0) where the streaming query will process data continuously instead of in micro-batches. While only a small subset of DataFrame operations allow this mode to be used, it can provide much lower latency (as low as milliseconds) than the micro-batch trigger modes.
                - Checkpoint location
                    - This is a directory in any HDFS-compatible filesystem where a streaming query saves its progress information—that is, what data has been successfully processed. Upon failure, this metadata is used to restart the failed query exactly where it left off. Therefore, setting this option is necessary for failure recovery with exactly-once guarantees.
        - Step 5: Start the query 

- Under the Hood of an Active Streaming Query
    - Once the query starts, the following sequence of steps transpires in the engine. The DataFrame operations are converted into a logical plan, which is an abstract representation of the computation that Spark SQL uses to plan a query:
    1. Spark SQL analyzes and optimizes this logical plan to ensure that it can be executed incrementally and efficiently on streaming data.
    2. Spark SQL starts a background thread that continuously executes the following loopThis execution loop runs for micro-batch-based trigger modes (i.e., ProcessingTime and Once), but not for the Continuous trigger mode.
        a. Based on the configured trigger interval, the thread checks the streaming sources for the availability of new data.
        b. If available, the new data is executed by running a micro-batch. From the optimized logical plan, an optimized Spark execution plan is generated that reads the new data from the source, incrementally computes the updated result, and writes the output to the sink according to the configured output mode.
        c. For every micro-batch, the exact range of data processed (e.g., the set of files or the range of Apache Kafka offsets) and any associated state are saved in the configured checkpoint location so that the query can deterministically reprocess the exact range if needed.
    3. This loop continues until the query is terminated, which can occur for one of the following reasons:
        a. A failure has occurred in the query (either a processing error or a failure in the cluster).
        b. The query is explicitly stopped using streamingQuery.stop().
        c. If the trigger is set to Once, then the query will stop on its own after executing a single micro-batch containing all the available data.

- Dynamic Partition Pruning
    - The idea behind dynamic partition pruning (DPP) is to skip over the data you don't need in a query's results. 
    - The typical scenario where DPP is optimal is when you are joining two tables: a fact table (partitioned over multiple columns) and a dimension table (nonpartitioned).
    - The key optimization technique in DPP is to take the result of the filter from the dimension table and inject it into the fact table as part of the scan operation to limit the data read.

- Adaptive Query Execution 
    - Adaptive Query Execution (AQE) reoptimizes and adjusts query plans based on runtime statistics collected in the process of query execution. It attempts to to do the following at runtime:
        • Reduce the number of reducers in the shuffle stage by decreasing the number of shuffle partitions.
        • Optimize the physical execution plan of the query, for example by converting a SortMergeJoin into a BroadcastHashJoin where appropriate.
        • Handle data skew during a join.

- The AQE framework
    - Spark operations in a query are pipelined and executed in parallel processes, but a shuffle or broadcast exchange breaks this pipeline, because the output of one stage is needed as input to the next stage. 
    - These breaking points are called materialization points in a query stage, and they present an opportunity to reoptimize and reexamine the query.
    - Here are the conceptual steps the AQE framework iterates over:
        1. All the leaf nodes, such as scan operations, of each stage are executed.
        2. Once the materialization point finishes executing, it's marked as complete, and all the relevant statistics garnered during execution are updated in its logical plan.
        3. Based on these statistics, such as number of partitions read, bytes of data read, etc., the framework runs the Catalyst optimizer again to understand whether it can:
            a. Coalesce the number of partitions to reduce the number of reducers to read shuffle data.
            b. Replace a sort merge join, based on the size of tables read, with a broadcast join.
            c. Try to remedy a skew join.
            d. Create a new optimized logical plan, followed by a new optimized physical plan.
        This process is repeated until all the stages of the query plan are executed.
    - Two Spark SQL configurations dictate how AQE will reduce the number of reducers:
        • spark.sql.adaptive.coalescePartitions.enabled (set to true)
        • spark.sql.adaptive.skewJoin.enabled (set to true)