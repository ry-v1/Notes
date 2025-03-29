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