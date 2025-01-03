This file has notes from the book "100 SQL Server Mistakes and How to Avoid Them"

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

- Development standards
  - Naming standards
    - Always use object names that are meaningful, with a view to making your code self-documenting.
    - Always consider database architecture, which is the design of your database, even within agile projects.
    - Avoid using prefixes for database objects, as they can actually make objects harder to locate.
    - Be especially careful to avoid using the sp_ prefix for stored procedures as this indicates that they are a system stored procedure, instead of a user-defined procedure.
  - Coding standards
    - Always make time to ensure that your data-tier application has coding standards. These should form part of your architectural efforts and should consider both stylistic choices as well as technical standards.
    - Avoid using ordinal column numbers
