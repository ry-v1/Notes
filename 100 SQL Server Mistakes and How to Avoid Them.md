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
