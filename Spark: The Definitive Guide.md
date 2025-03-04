# Spark: The Definitive Guide

- Spark Applications consist of a driver process and a set of executor processes. The driver process runs your main() function, sits on a node in the cluster, and is responsible for three things:
    - maintaining information about the Spark Application; 
    - responding to a user's program or input;
    - and analyzing, distributing, and scheduling work across the executors (discussed momentarily).

- The driver process is absolutely essential—it's the heart of a Spark Application and maintains all relevant information during the lifetime of the application.

- The executors are responsible for actually carrying out the work that the driver assigns them. This means that each executor is responsible for only two things: 
    - executing code assigned to it by the driver, 
    - and reporting the state of the computation on that executor back to the driver node.

- Starting Spark
    - When you start Spark in this interactive mode, you implicitly create a SparkSession that manages the Spark Application. 
    - When you start it through a standalone application, you must create the SparkSession object yourself in your application code.

- The SparkSession
    - The SparkSession instance is the way Spark executes user-defined manipulations across the cluster. 
    - There is a one-to-one correspondence between a SparkSession and a Spark Application.

- DataFrames
    - A DataFrame is the most common Structured API and simply represents a table of data with rows and columns. 
    - The list that defines the columns and the types within those columns is called the schema. 

- Partitions
    - To allow every executor to perform work in parallel, Spark breaks up the data into chunks called partitions. 
    - A partition is a collection of rows that sit on one physical machine in your cluster. 
    - A DataFrame's partitions represent how the data is physically distributed across the cluster of machines during execution.

- Transformations
    - Transformations are the core of how you express your business logic using Spark. 
    - There are two types of transformations: those that specify narrow dependencies, and those that specify wide dependencies.
    - Transformations consisting of narrow dependencies (we’ll call them narrow transformations) are those for which each input partition will contribute to only one output partition.
    - A wide dependency (or wide transformation) style transformation will have input partitions contributing to many output partitions. 
    - You will often hear this referred to as a shuffle whereby Spark will exchange partitions across the cluster. 
    - With narrow transformations, Spark will automatically perform an operation called pipelining, meaning that if we specify multiple filters on DataFrames, they’ll all be performed in-memory. 
    - The same cannot be said for shuffles. When we perform a shuffle, Spark writes the results to disk.

- Lazy Evaluation
    - Lazy evaulation means that Spark will wait until the very last moment to execute the graph of computation instructions. 
    - In Spark, instead of modifying the data immediately when you express some operation, you build up a plan of transformations that you would like to apply to your source data. 
    - By waiting until the last minute to execute the code, Spark compiles this plan from your raw DataFrame transformations to a streamlined physical plan that will run as efficiently as possible across the cluster. 
    - This provides immense benefits because Spark can optimize the entire data flow from end to end. 
    - An example of this is something called predicate pushdown on DataFrames. 
    - If we build a large Spark job but specify a filter at the end that only requires us to fetch one row from our source data, the most efficient way to execute this is to access the single record that we need. 
    - Spark will actually optimize this for us by pushing the filter down automatically.

- Actions
    - Transformations allow us to build up our logical transformation plan. To trigger the computation, we run an action. 
    - An action instructs Spark to compute a result from a series of transformations.

- Spark UI
    - You can monitor the progress of a job through the Spark web UI. The Spark UI is available on port 4040 of the driver node. 
    - If you are running in local mode, this will be http://localhost:4040.
    - The Spark UI displays information on the state of your Spark jobs, its environment, and cluster state. It’s very useful, especially for tuning and debugging.

- Overview of Structured API Execution
    1. Write DataFrame/Dataset/SQL Code.
    2. If valid code, Spark converts this to a Logical Plan.
    3. Spark transforms this Logical Plan to a Physical Plan, checking for optimizations along the way.
    4. Spark then executes this Physical Plan (RDD manipulations) on the cluster.

- Logical Planning
    - This logical plan only represents a set of abstract transformations that do not refer to executors or drivers, it’s purely to convert the user’s set of expressions into the most optimized version. It does this by converting user code into an unresolved logical plan. 
    - This plan is unresolved because although your code might be valid, the tables or columns that it refers to might or might not exist. 
    - Spark uses the catalog, a repository of all table and DataFrame information, to resolve columns and tables in the analyzer. 
    - The analyzer might reject the unresolved logical plan if the required table or column name does not exist in the catalog. 
    - If the analyzer can resolve it, the result is passed through the Catalyst Optimizer, a collection of rules that attempt to optimize the logical plan by pushing down predicates or selections. 
    - Packages can extend the Catalyst to include their own rules for domain-specific optimizations.

- Physical Planning
    - The physical plan, often called a Spark plan, specifies how the logical plan will execute on the cluster by generating different physical execution strategies and comparing them through a cost model.

- Schemas
    - A schema defines the column names and types of a DataFrame. We can either let a data source define the schema (called schema-on-read) or we can define it explicitly ourselves.
    - A schema is a StructType made up of a number of fields,
        - StructFields, that have a name, type, 
        - a Boolean flag which specifies whether that column can contain missing or null values, and, 
        - users can optionally specify associated metadata with that column.

- Columns
    - There are a lot of different ways to construct and refer to columns but the two simplest ways are by using the col or column functions.
    - printSchema : to see a DataFrame's columns
    - columns : property to programmatically access columns

- Expressions
    - An expression is a set of transformations on one or more values in a record in a DataFrame.

- Repartition
    - Repartition will incur a full shuffle of the data, regardless of whether one is necessary. 
    - This means that you should typically only repartition when the future number of partitions is greater than your current number of partitions or when you are looking to partition by a set of columns.

- Coalesce
    - Coalesce will not incur a full shuffle and will try to combine partitions.

- How Spark Performs Joins 
    - To understand how Spark performs joins, you need to understand the two core resources at play: the node-to-node communication strategy and per node computation strategy.
    - Spark approaches cluster communication in two different ways during joins. It either incurs a shuffle join, which results in an all-to-all communication or a broadcast join.

    - When you join a big table to another big table, you end up with a shuffle join.
    - In a shuffle join, every node talks to every other node and they share data according to which node has a certain key or set of keys (on which you are joining). 
    - These joins are expensive because the network can become congested with traffic, especially if your data is not partitioned well.

    - When the table is small enough to fit into the memory of a single worker node, with some breathing room of course, we can optimize our join. Although we can use a big table-to-big table communication strategy, it can often be more efficient to use a broadcast join. 
    - What this means is that we will replicate our small DataFrame onto every worker node in the cluster (be it located on one machine or many). 
    - Now this sounds expensive. However, what this does is prevent us from performing the all-to-all communication during the entire join process. 
    - Instead, we perform it only once at the beginning and then let each individual worker node perform the work without having to wait or communicate with any other worker node
    - At the beginning of this join will be a large communication. However, immediately after that first, there will be no further communication between nodes.
    - This means that joins will be performed on every single node individually, making CPU the biggest bottleneck. 

    - When performing joins with small tables, it's usually best to let Spark decide how to join them. You can always force a broadcast join if you're noticing strange behavior.

- Spark-Managed Tables
    - Tables store two important pieces of information. The data within the tables as well as the data about the tables; that is, the metadata. 
    - You can have Spark manage the metadata for a set of files as well as for the data. 
    - When you define a table from files on disk, you are defining an unmanaged table. 
    - When you use saveAsTable on a DataFrame, you are creating a managed table for which Spark will track of all of the relevant information.

- Distributed Shared Variables
    - Broadcast Variables
        - Broadcast variables are a way you can share an immutable value efficiently around the cluster without encapsulating that variable in a function closure.
        - Broadcast variables are shared, immutable variables that are cached on every machine in the cluster instead of serialized with every single task.
    - Accumulators
        - Accumulators are a way of updating a value inside of a variety of transformations and propagating that value to the driver node in an efficient and fault-tolerant way.
        - For accumulator updates performed inside actions only, Spark guarantees that each task's update to the accumulator will be applied only once, meaning that restarted tasks will not update the value. 
        - In transformations, you should be aware that each task's update can be applied more than once if tasks or job stages are reexecuted.

- Execution Modes
    - Cluster mode
        - In cluster mode, a user submits a pre-compiled JAR, Python script, or R script to a cluster manager. 
        - The cluster manager then launches the driver process on a worker node inside the cluster, in addition to the executor processes. 
        - This means that the cluster manager is responsible for maintaining all Spark Application-related processes.
    - Client mode
        - Client mode is nearly the same as cluster mode except that the Spark driver remains on the client machine that submitted the application. 
        - This means that the client machine is responsible for maintaining the Spark driver process, and the cluster manager maintains the executor processses.
    - Local mode
        - Local mode is a significant departure from the previous two modes: it runs the entire Spark Application on a single machine. 
        - It achieves parallelism through threads on that single machine.
        - This is a common way to learn Spark, to test your applications, or experiment iteratively with local development.

- The Life Cycle of a Spark Application (Outside Spark)
    - Client Request
        - The first step is for you to submit an actual application. This will be a pre-compiled JAR or library. 
        - At this point, you are executing code on your local machine and you’re going to make a request to the cluster manager driver node.
        - Here, we are explicitly asking for resources for the Spark driver process only. 
        - We assume that the cluster manager accepts this offer and places the driver onto a node in the cluster. 
        - The client process that submitted the original job exits and the application is off and running on the cluster.

    - Launch
        - Now that the driver process has been placed on the cluster, it begins running user code
        - This code must include a SparkSession that initializes a Spark cluster (e.g., driver + executors). 
        - The SparkSession will subsequently communicate with the cluster manager, asking it to launch Spark executor processes across the cluster. 
        - The number of executors and their relevant configurations are set by the user via the command-line arguments in the original spark-submit call.
    
    - Execution
        - Now that we have a “Spark Cluster, ” Spark goes about its merry way executing code. 
        - The driver and the workers communicate among themselves, executing code and moving data around. 
        - The driver schedules tasks onto each worker, and each worker responds with the status of those tasks and success or failure.
    
    - Completion
        - After a Spark Application completes, the driver processs exits with either success or failure. 
        - The cluster manager then shuts down the executors in that Spark cluster for the driver. 
        - At this point, you can see the success or failure of the Spark Application by asking the cluster manager for this information.

- The Life Cycle of a Spark Application (Inside Spark)
    -The SparkSession
        - The first step of any Spark Application is creating a SparkSession. 
        - In many interactive modes, this is done for you, but in an application, you must do it manually.