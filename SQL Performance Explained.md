# SQL Performance Explained

### Anatomy of an Index

    - An index is a distinct structure in the database that is built using the create index statement. 
    - It requires its own disk space and holds a copy of the indexed table data. That means that an index is pure redundancy.
    - Creating an index does not change the table data; it just creates a new data structure that refers to the table.

    - An SQL database must process insert, delete and update statements immediately, keeping the index order without moving large amounts of data.
    - The database combines two data structures to meet the challenge: a doubly linked list and a search tree. These two structures explain most of the database’s performance characteristics.

##### The Index Leaf Nodes

    - Databases use doubly linked lists to connect the so-called index leaf nodes.
    - Each leaf node is stored in a database block or page; that is, the database’s smallest storage unit. 
    - All index blocks are of the same size—typically a few kilobytes. 
    - The database uses the space in each block to the extent possible and stores as many index entries as possible in each block. That means that the index order is maintained on two different levels: the index entries within each leaf node, and the leaf nodes among each other using a doubly linked list.

##### The Search Tree (B-Tree)
    - The index leaf nodes are stored in an arbitrary order—the position on the disk does not correspond to the logical position according to the index order.
    - A database needs a second structure to find the entry among the shuffled pages quickly: a balanced search tree in short: the B-tree.
    - A B-tree is a balanced tree—not a binary tree.

##### Slow Indexes, Part I
    - The first ingredient for a slow index lookup is the leaf node chain.
    - The database must read the next leaf node to see if there are any more matching entries. That means that an index lookup not only needs to perform the tree traversal,it also needs to follow the leaf node chain.
    - The second ingredient for a slow index lookup is accessing the table.
    - Even a single leaf node might contain many hits—often hundreds. The corresponding table data is usually scattered across many table blocks. That means that there is an additional table access for each hit.

    - An index lookup requires three steps: (1) the tree traversal; (2) following the leaf node chain; (3) fetching the table data. The tree traversal is the only step that has an upper bound for the number of accessed blocks—the index depth. The other two steps might need to access many blocks—they cause a slow index lookup.

### The Where Clause

##### Primary Keys
    - The database automatically creates an index for the primary key.
    - The where clause cannot match multiple rows because the primary key constraint ensures uniqueness of the key values. The database does not need to follow the index leaf nodes—it is enough to traverse the index tree.
    - A primary key does not necessarily need a unique index—you can use a non-unique index as well.
    - One of the reasons for using non-unique indexes for a primary keys are deferrable constraints. As opposed to regular constraints, which are validated during statement execution, the database postpones the validation of deferrable constraints until the transaction is committed. Deferred constraints are required for inserting data into tables with circular dependencies.
    