# SQL Performance Explained

### Anatomy of an Index

- An index is a distinct structure in the database that is built using the create index statement. 
- It requires its own disk space and holds a copy of the indexed table data. That means that an index is pure redundancy.
- Creating an index does not change the table data; it just creates a new data structure that refers to the table.

- An SQL database must process insert, delete and update statements immediately, keeping the index order without moving large amounts of data.
- The database combines two data structures to meet the challenge: a doubly linked list and a search tree. These two structures explain most of the database’s performance characteristics.
- Databases use doubly linked lists to connect the so-called index leaf nodes.
- Each leaf node is stored in a database block or page; that is, the database’s smallest storage unit. 
- All index blocks are of the same size—typically a few kilobytes. 
- The database uses the space in each block to the extent possible and stores as many index entries as possible in each block. That means that the index order is maintained on two different levels: the index entries within each leaf node, and the leaf nodes among each other using a doubly linked list.
