# Spark: The Definitive Guide

- Spark Applications consist of a driver process and a set of executor processes. The driver process runs your main() function, sits on a node in the cluster, and is responsible for three things:
    - maintaining information about the Spark Application; 
    - responding to a user’s program or input;
    - and analyzing, distributing, and scheduling work across the executors (discussed momentarily).

- The driver process is absolutely essential—it’s the heart of a Spark Application and maintains all relevant information during the lifetime of the application.

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
    - A DataFrame’s partitions represent how the data is physically distributed across the cluster of machines during execution.

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