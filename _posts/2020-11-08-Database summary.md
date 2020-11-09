# Database system summary

##  Different design for data model

### Hierarchical structure

Records' relationships form a treelike model, which is simple but inflexible because the relationship is confined to a one-to-many relaitonship.

The main drawback of hierarchical structure is :

1. Information is repeated.
2. Existence depends on parents.
3. Record-at-a-time access DML

### Network structure

Networks are more flexible than hierarchies but more complex.

Loading and recovering networks is more complex than hierarchies

### Relational

设计理念:

1. Store the data in a simple data structure (Table)
2. Access it through a high level set-at-a-time DML
3. No need for a physical storage proposal

进一步产生了 -> Entity-relationship (ER) data model

Statements:

A data base be thought of a collection of instances of entities. In addition, entities have attributes, which are the data elements that characterize the entity. One or more of there atrributes would be designated to be unique (key). Lastly, there could be relationships between the entities. Relationships could be 1-to-1, 1-to-n, n-to-1 or m-to-n, depending on how the entities participate in the relationship.

## Concepts

A data model is collection of concepts for describing the data in a database

A schema is a description of a particular collection of data, using a given data model

## Data model

Relational

NoSQL:

​	Key/Value

​	Graph

​	Column-family

## Relational model

Primary key: Uniquely identifies a single tuple

Auto-generation of unique integer:

MySQL -> AUTO-INCREMENT

Foreign key: specifies that an attribute from one relation has to map to a tuple in another relation

## Relational algebra

Operators: 

![image-20201108142657629](/images/image-20201108142657629.png)

Each operator takes one or more relations as inputs and outputs a new relation

### Select

Choose a subset of tuples from a relation that satisfies a selection predicate.

-> Predicate acts as a filter to retain only tuples that fullfill it qualifying requirement.

-> Can combine multiple predicates using conjunction / disjunction

### Projection

Generate a relation with tuples that contains only the specified attributes. 

-> Can rearrange attributes' ordering

-> Can manipulate the value

### Union 

Generate a relation that contains all tuples that appear in either only one or both input relation. 

Intersection similar

### Different

Generation a relation that contains only the tuples that appear in the first and not the second of the input relations

### Product (Cross Join)

Generate a relation that contains all possible combinations of tuples from the input relations.

### Join

Generate a relation that cantains all tuples that are a combination of two tuples, with a common values for one or more attributes.

### Extra operatiors

Rename, Assignment, Duplicate Elimination, Aggregation, Sorting, Division

## SQL

Relational algebra defines the high-level steps of how to compute a query

![image-20201108144020613](/images/image-20201108144020613.png)



A better approach is to state the high-lelve answer that you want the DBMS to compute. SQL is the de facto standard

Including :
Data Manipulation Language (DML)

Data Definition Language (DDL)

Data Control Language (DCL)

View definition

Integrity & Referential constrains

Transaction

**Important**: SQL is based on bags(duplicates) not sets(no duplicates)

### Example Database

![image-20201108161903848](/images/image-20201108161903848.png)

### Aggregates

Functions that return a single value from a bag of tuples

Aggregate functions can only be used in the SELECT output list

```SQL
SELECT COUNT(login) AS cnt FROM student WHERE login LIKE '%@cs'
```

COUNT, SUM, AVG support DISTINCT

Output of other columns outside of an aggregate is undefined

### Group By

Project tuples into subsets and calculate aggregates against each subset.

![image-20201108161636721](/images/image-20201108161636721.png)

![image-20201108161648287](/images/image-20201108161648287.png)

![image-20201108161741616](/images/image-20201108161741616.png)

Non-aggregated values in SELECT output clause must appear in GROUP BY clause.

![image-20201108161835568](/images/image-20201108161835568.png)

### Having

Filter results based on aggregation computation. Like a WHERE clause for a GROUP BY

![image-20201108162207255](/images/image-20201108162207255.png)

上面这个SQL语句是错误的，因为SQL的执行顺序是

$FROM \rightarrow WHERE \rightarrow GROUP \ BY \rightarrow AGGREGATION \rightarrow HAVING$

所以当WHERE clause执行时，并不知道avg_gpa的值。这是就要用到having 语句， 当aggregation根据group by执行完后，使用Having进行filtering 操作。

![image-20201108162512711](/images/image-20201108162512711.png)

![image-20201108162528427](/images/image-20201108162528427.png)

### String operations

**Like** is used for string matching

'%' Matches any substring(including empty strings)

'_' Match any one character

![image-20201108163626826](/images/image-20201108163626826.png)

![image-20201108163638836](/images/image-20201108163638836.png)



|| : concatenate two or more strings togather

![image-20201108164143229](/images/image-20201108164143229.png)

### Output Redirection

Store query results in another table, table will have the same number of columns with the same types as the input

![image-20201108164337443](/images/image-20201108164337443.png)

### Output control

```SQL
ORDERED BY <column*> [ASC|DESC]
```

Order the output tuples by the values in one or more of their columns

```sql
LIMIT <count> [OFFSET]
```

-> Limit the number of tuples returned in output

-> Can set an offset to return a "range"

### Nested queries

Queries containing other queries. They are often difficult to optimize.

Inner queries can appear (almost) angwhere in query

ALL -> Must satisfy expression for all rows in subquery

ANY-> Must satisfy expression for at least one row in sub-query

IN -> Equivalent to '=ANY()'

EXISTS -> At least one row is returned.

![image-20201108170028743](/images/image-20201108170028743.png)

![image-20201108170137482](/images/image-20201108170137482.png)

![image-20201108170151568](/images/image-20201108170151568.png)

![image-20201108170259747](/images/image-20201108170259747.png)

### Window functions

Performs a "sliding" calculation across a set of tuples that are related.

![image-20201108170423991](/images/image-20201108170423991.png)

![image-20201108170503168](/images/image-20201108170503168.png)

Aggregation functions:

Special window functions:
-> ROW_NUMBER() -> number of the current row

-> Rank() -> Order position of the current row

![image-20201108170702665](/images/image-20201108170702665.png)

The **OVER** keyword specifies how to group together tuples when computing the window function.

Use **Partition by** to specify group.

![image-20201108170902472](/images/image-20201108170902472.png)

![image-20201108170913991](/images/image-20201108170913991.png)

You can also include an **ORDER BY** in the window grouping to sort entries in each group

![image-20201108171205955](/images/image-20201108171205955.png)

Group tuples by cid then sort by grade

### COMMON TABLE EXPRESSIONS

Provides a way to write auxiliary statements for use in a larger query

-> Think of it like a temp table just for one query

Alternative to nested queries and views.

![image-20201108171351321](/images/image-20201108171351321.png)

You can bind output columns to names before the AS keyword

![image-20201108171444749](/images/image-20201108171444749.png)

## Storage

### Access Time

![image-20201108185241648](/images/image-20201108185241648.png)

### 为什么不借助操作系统

One can use memory mapping(mmap) to store the contents of a file into a process' address space.

The OS is responsible for moving data for moving the files' pages in and out of memory.

DBMS (almost) always want to control things itself and can do a better job at it.

-> Flushing dirty pages to disk in the correct order

-> Specialized prefetching

-> Buffer replacement policy

-> Thread/process scheduling

### File storage

The DBMS stores a database as one or more files on disk. The os doesn't know anything about the contents of these files.

#### Storage manager

The storage manager is responsible for maintaining a database's files. Some do their own scheduling for reads and writes to improve spatial and temporal locality of pages.

It organizes the files as a collection of *pages*.

-> Track data read/written to pages.

-> Track the available space

A page is a fixed-size block of data.

-> It can contain tuples, meta-data, indexes, log records..

-> Most systems do not mix page types

-> Some systems require a page to be self-contained.

Each page is given a unique identifier.

-> The DBMS uses an indirection layer to map page ids to physical locations.

Different DBMSs manage pages in files on disk in different ways

-> Heap file organization

-> Sequential/ Sorted File Organization

-> Hashing File Organization

#### DATABASE HEAP

A heap file is an unordered collection of pages .

Need meta-data to keep track of what pages exist and which ones have free space.

Two ways to represent a heap file:

-> LinkedList

-> Page directory

#### Linked list

![image-20201108192652451](/images/image-20201108192652451.png)

Maintain a header page at the begining of the file that stores two pointers:

-> HEAD of the free page list

-> HEAD of the data page list

Each page keeps track of the number of free slots in itselt

#### Page directory

The DBMS maintains special pages that tracks the location of data pages in the database files.

The directory also records the number of free slots per page

The DBMS has to make sure that the directory pages are in sync with data pages.

#### Page Header

Every page contains a header of meta-data about the page's contents.

-> Page size

-> Checksum

-> DBMS Version

-> Transaction Visibility

-> Compression information

### Page Layout

So how to organize the data stored inside of the page?

Two approaches:

1. Tuple-oriented
2. Log-structured

#### Slotted Pages

![image-20201108193325604](/images/image-20201108193325604.png)

The Slot Array maps "slots" to the tuples' starting position offsets.

The header keeps track of:

1. The number of used slots
2. The offset of the starting location of the last slot used.

#### Log-structed file organization

Instead of storing tuples in pages, the DBMS only sotres log records.

The system appends log records to the file of how the database was modified.

-> Inserts store the entire tuple

-> Deletes mark the tuple as deleted

-> Updates contain the delta of just the attributes that were modified.

![image-20201108193648261](/images/image-20201108193648261.png)

To read a record, The DBMS scans the log backwards and "recreates" the tuple to find what it needs.

![image-20201108193700044](/images/image-20201108193700044.png)

Build indexes to allow it to jump to locations in the log

![image-20201108193815363](/images/image-20201108193815363.png)

Periodically compact the log

### Tuple Layout

A tuple is essentially a sequence of bytes. It is the job of DBMS to interpret those bytes into attributes types and values.

#### Tuple header

Each tuple is prefixed with a header that contains meta-data about it.

-> Visibility info(Concurrency control)

-> Bit Map for NULL values

#### Record IDs

The DBMS needs a way to keep track of individual tuples. Each tuple is assigned a unique record identifier.

-> Most common: page_id + offset/slot

-> Can also contain file location info

### System catalog

A DBMS stores meta-data about databases in its internal catalogs

-> Tables, Columns, indexes, views

-> Users, Permissions

-> Internal statistics

Accessing table schema: 

```MYSQL
DESCRIBE student; 
```

![image-20201108200912887](/images/image-20201108200912887.png)

### OLTP: On-line transaction processing

Simple queries that read/update a small amount of data that is related to a single entity in the database.

### OLAP: On-Line Analytical Processing

Complex queries that read large portions of the database spanning multiple entities.

## Buffer pool

How the DBMS manages its memory and move data back-and-forth from disk

Spatial Control:
-> Where to write pages on disk;

-> **The goal is to keep pages that are used together often as physically close together as possible on disk.**

Temporal Control:

-> When to read pages into memory, and when to write them to disk.

-> **The goal is minimize the number of stalls from having to read data from disk.**

### Buffer pool organization

Memory region organized as an array of fixed-size pages.

An array entry is called a frame

When the DBMS requests a page, an exact copy is placed into one of these frames.

![image-20201108213556199](/images/image-20201108213556199.png)

The **page table** keeps track of pages that are currently in memory

Also maintains additional meta-data per page:

-> **Dirty Flag**

-> **Pin/Reference Counter**

![image-20201108213726867](/images/image-20201108213726867.png)

If page3 is fetched by the DBMS, then this page thould be pinned, so it can not be replaced by replacement policy.

![image-20201108213821530](/images/image-20201108213821530.png)

### Lock verse Latches

Locks:

1. Protects the database's logical contents from other transactions
2. Held for transaction duration
3. Need to be able to rollback changes

Latches:

1. Protects the critical sections of DBMS's internal data structure from other threads.
2. Held for operation duration.
3. Do not need to be able to rollback changes.

### Page table verse Page directory

The page directory is the mapping from the page ids to page location in the database files.

-> All changes must be recorded on disk to allow the DBMS to find on restart.

The page table is the mapping from the page ids to to a copy of the page in buffer pool frames.

-> This is an in-memory data structure that does not need to be store on disk

### Buffer pool optimizations

#### Multiple buffer pools

The DBMS does not always have a single buffer pool for the entire system.

-> Multiple buffer pool instances.

-> Per-database buffer pool

-> Per-page type buffer pool

Helps reduce latch contention and improve locality

#### Pre-fetching

The DBMS can also prefetch pages based on a query plan.

-> Sequential scans

-> Index scans

```mysql
SELECT * FROM A WHERE val BETWEEN 100 AND 250
```

![image-20201108214854953](/images/image-20201108214854953.png)

![image-20201108214912333](/images/image-20201108214912333.png)

![image-20201108214930838](/images/image-20201108214930838.png)

So we can prefetch the index-page5 in the same time we fetch the page index-page3.

#### Scan sharing

Queries can reuse data retrieved from storage or operator computations.

Allow multiple queries to attach to a single cursor that scans a tables.

-> Queries do not have to be the same 

-> Can also share intermediate results

If a query starts a scan and if there one already doing this, then the DBMS will attach to the second query's cursor.

-> The DBMS keep track of where the second query joined with the first so that it can finish the scan when it reaches the end of the data structure.

### Buffer pool bypass

The sequential scan operator will not store fetched pages in the buffer pool to avoid overhead.

-> Memory is local to running query.

-> Works well if operator needs to read a large sequence of pages that are contiguous on disk.

### OS page cache

Most disk operations go through the OS API.

Unless you tell it not to, the OS maintains its own filesystem cache.

Most DBMSs use direct I/O (O_DIRECT) to bypass the OS cache.

-> Redundant copies of pages.

-> Different eviction policies.

### Buffer replacement policies

When the DBMS needs to free up a frame to make room for a new page, it must decide which page to evict from the buffer pool.

-> LRU

-> Clock: Approximation of LRU without needing a separate timestamp per page.

​	-> Each page has a reference bit

​	-> When a page is accessed, set to 1

Organize the pages in a circular buffer with a "clock hand"

-> Upon sweeping, check if a page's bit is set to 1

-> If yes, set to 0, if no, then evict

LRU and CLOCK replacement policies are susceptible to sequential flooding. → A query performs a sequential scan that reads every page. → This pollutes the buffer pool with pages that are read once and then never again.

The most recently used page is actually the most unneeded page.

#### BETTER POLICIES: LRU-K

Track the history of the last K references as timestamps and compute the interval between subsequent accesses.

The DBMS then uses this history to estimate the next time that page is going to be accessed.

#### BETTER POLICIES: LOCALIZATION

The DBMS chooses which pages to evict on a per txn/query basis. This minimizes the pollution of the buffer pool from each query. 

→ Keep track of the pages that a query has accessed.

### Dirty pages

Fast: If a page in the buffer pool is not dirty, the DBMS can simple "drop" it.

SLOW: if a page is dirty, then the DBMS must write back to disk to ensure that its changes are persisted.

Trade-off between fast evictions versus dirty writing pages that will not be read again in the future.

The DBMS can periodically walk through the page table and write dirty pages to disk. 

When a dirty page is safely written, the DBMS can either evict the page or just unset the dirty flag. 

Need to be careful that we don’t write dirty pages before their log records have been written…

### Other memory pools

The DBMS needs memory for things other than just tuples and indexes. 

These other memory pools may not always backed by disk. Depends on implementation.

-> Sorting + Join buffers

-> Query Caches

-> Maintenance Buffers

-> Log Buffers

-> Dictionary Caches

## HASH TABLES

a **hash table** implements an unordered associative array that maps keys to values.

It uses a hash function to compute an offset into the array for a given key, from which the desired value can be found

Space Complexity: $O(n)$

Operation Complexity:

-> Average: $O(1)$  Money cares about constants

-> Worst: $O(n)$

### STATIC HASH TABLE

allocate a giant array that has one slot for every element you need to store.

To find an entry, mod the key by the number of elements to find the offset in the array.

### Assumption

You know the number of elements ahead of time.

Each key is unique.

Prefect hash function: if key1 != key2, then hash(key1) != hash(key2)

### Design Decision

1. Hash function

   -> How to map a large key space into a small domain

   -> Trade-off between being fast vs. collision rate

2. Hashing Schema

   -> How to handle key collisions after hashing

   -> Trade-off between allocating a large hash table vs. additional instructions to find/insert keys.

### HASH FUNCTIONS

For any input key, return an integer representation of that key

we want something that is fast and has a slow collision rate

[CRC-64(1975)](https://create.stephan-brumme.com/crc32/)

-> Used in networking for error detection.

[MurmurHash(2008)](https://github.com/aappleby/smhasher)

-> Designed to a fast, general purpose hash function

[Google CityHash(2011)](https://github.com/google/cityhash)

-> Design to be faster for short keys (<64 bytes).

[FaceBook XXHash(2012)](http://cyan4973.github.io/xxHash/)

-> From the creator of zstd compression.

[Google FarmHash(2014)](https://github.com/google/farmhash)

->Newer version of CityHash with better collision rates

![image-20201109094223095](/images/image-20201109094223095.png)

<img src="/images/image-20201109094245099.png" alt="image-20201109094245099" style="zoom: 50%;" />

## Static Hashing Schemas

### Approach 1: Linear Probe Hashing

Single giant table of slots

Resolve collisions by linearly searching for the next free slot in the table.

​	-> To determine whether an element is present, hash to a location in the index and scan for it.

​	-> Have to store the key in the index to know when to stop scanning

​	-> Insertions and deletions are generalizations of lookups

Delete:

​	-> approach 1: if a element in the hash table is deleted, mark it as "Tombstone".

​	-> approach 2: movement

#### Non-unique keys

choice 1: Separate Linked List

​	-> Store values in separate storage area for each key

![image-20201109095036835](/images/image-20201109095036835.png)

 Choice 2: Redundant keys

​	-> Store duplicate keys entries together in the hash table

![image-20201109095127549](/images/image-20201109095127549.png)

### Robin hood hashing

Variant of linear probe hashing that steals slots from **Rich** keys and give them to **Poor** keys

-> Each key tracks the number of positions they are from where its optimal position in the table.

-> On insert, a key takes the slot of another key if the first key is farther away from its optimal position than the second key.

Example:

![image-20201109095412909](/images/image-20201109095412909.png)

![image-20201109095426183](/images/image-20201109095426183.png)

![image-20201109095440424](/images/image-20201109095440424.png)

![image-20201109095455799](/images/image-20201109095455799.png)

![image-20201109095509196](/images/image-20201109095509196.png)

![image-20201109095527294](/images/image-20201109095527294.png)

![image-20201109095542479](/images/image-20201109095542479.png)

### Cuckoo Hashing

Use multiple hash tables with different hash function seeds.

-> On insert, check every table and pick anyone that has a free slot

-> If no tables has a free slot, evict the element from one of them and then re-hash it find a new location

Look-ups and deletions are always $O(1)$ because only one location per hash is checked.

![image-20201109100915526](/images/image-20201109100915526.png)

![image-20201109100926096](/images/image-20201109100926096.png)

![image-20201109100938116](/images/image-20201109100938116.png)

![image-20201109100949124](/images/image-20201109100949124.png)

![image-20201109100959994](/images/image-20201109100959994.png)

![image-20201109101013209](/images/image-20201109101013209.png)

![image-20201109101024973](/images/image-20201109101024973.png)

![image-20201109101035177](/images/image-20201109101035177.png)

## Obeservation

The previous hash tables require the DBMS to know the number of elements it wants to store.

-> Otherwise it has rebuild the table if it need grow/shrink in size

Dynamic hash tables resize themselves on demand

-> Chained Hashing

-> Extendible Hashing

-> Linear Hashing

### Chained Hashing

Maintain a linked list of **buckets** for each slot in the hash table.

Resolve collisions by placing all elements with the same hash key into the same bucket.

-> To determine whether an element is present, hash to its bucket and scan for it.

-> Insertions and deletions are generalizations of lookups.

![image-20201109102014723](/images/image-20201109102014723.png)

### Extendible Hashing

Chained-hashing approach where we split buckets instead of letting the linked list grow forever.

Multiple slot locations can point to the same bucket chain.

Reshuffling bucket entries on splitand increase the number of bits to examine.

-> Data movement is localized to just the split chain.

![image-20201109102221657](/images/image-20201109102221657.png)

![image-20201109102241555](/images/image-20201109102241555.png)

because the global bit number is 2, we compare the first 2 bits of hash(A) within the bit array

![image-20201109102340112](/images/image-20201109102340112.png)

![image-20201109102354770](/images/image-20201109102354770.png)

这时，(10) slot对应的bucket已经满了，同时这个bucket的local bit number是2， 等于global bit number, 所以global bit number加1

![image-20201109102516065](/images/image-20201109102516065.png)

并重新分配bit array

![image-20201109102529981](/images/image-20201109102529981.png)

![image-20201109102626557](/images/image-20201109102626557.png)

![image-20201109102647339](/images/image-20201109102647339.png)

### Linear Hashing 

The hash table maintains a pointer that tracks the next bucket to split

-> When any bucket overflows, split the bucket at the pointer location

Use multiple hashes to find the rigth bucket for a given key.

![image-20201109102918738](/images/image-20201109102918738.png)

![image-20201109102929735](/images/image-20201109102929735.png)

![image-20201109102944526](/images/image-20201109102944526.png)

![image-20201109103003030](/images/image-20201109103003030.png)

![image-20201109103014455](/images/image-20201109103014455.png)

![image-20201109103030655](/images/image-20201109103030655.png)

![image-20201109103053856](/images/image-20201109103053856.png)

![image-20201109103111196](/images/image-20201109103111196.png)

![image-20201109103122854](/images/image-20201109103122854.png)

![image-20201109103133641](/images/image-20201109103133641.png)

![image-20201109103149184](/images/image-20201109103149184.png)

Splitting buckets based on the split pointer will eventually get to all overflowed buckets.

-> When the pointer reaches the last slot, delete the first hash fuction and move back the beginning.

![image-20201109103332581](/images/image-20201109103332581.png)

![image-20201109103406238](/images/image-20201109103406238.png)

![image-20201109103446689](/images/image-20201109103446689.png)

Question 1: Cuckoo Hashing

Consider the following cuckoo hashing schema:

1. Both tables have a size of 4.
2. The hashing function of the first table returns the lowest two bits: $h_1(x) = x \& 0b11$
3. The hashing function of the second table returns the next two bits: $h_2(x) = (x >> 2) \& 0b11$
4. When replacement is necessary, first select an element in the second table.
5. The original content is show int Figure 1.

![image-20201109104507739](/images/image-20201109104507739.png)

(a) Insert keys 12 and 10, Draw the resulting two tables.

$h_1(12) = 0$

$h_2(12) = 3$

$h_1(10) = 2$

$h_2(10) = 2$

<img src="/images/image-20201109105036203.png" alt="image-20201109105036203" style="zoom:50%;" />

(b) Then delete 14, and insert 8, draw the resulting two tables.

$h_1(8) = 0$

$h_2(8)=2$

rehash 10

$h_1(10) = 2$

$h_2(10) = 2$

<img src="/images/image-20201109105911121.png" alt="image-20201109105911121" style="zoom:50%;" />

(c) Finally, insert 28, draw the resulting two tables

$h_1(28) = 0$ 

$h_2(28) = 3$

rehasing 12

$h_1(12)=0$

$h_2(12)=3$

rehasing 4

$h_1(4) = 0$

$h_2(4) = 1$

<img src="/images/image-20201109105832826.png" alt="image-20201109105832826" style="zoom:50%;" />

Question 2: Extendible Hashing

Consider an extendible hashing structure such that:

Each bucket can hold up to two records

The hashing function uses the lowest g bits, where g is the globale depth

(a) starting from an empty table, insert keys 15, 3, 7, 14

1. What is the global depth of the resulting table? **3**
2. What is the local depth the bucket containing 14? **1**
3. What is the local depth of the bucket containing 3? **3**

(b) Starting from the result in (a), you insert keys 1, 9, 23, 11, 17

 	1. Which key will first cause a split (without doubling the size of the table)? **17**
 	2. Which key will first make the table double in size?  **23**

(c) Now consider the table below, along with the following deletion rules:

 	1. If two buckets have the same local depth d, and share the first d − 1 bits of their indexes (e.g. 010 and 110 share the first 2 bits), then they can be merged if the total capacity fits in a single bucket. The resulting local depth is d − 1.
 	2. If the global depth g becomes strictly greater than all local depths, then the table can be halved in size. The resulting global depth is g − 1

![image-20201109112410944](/images/image-20201109112410944.png)

Starting from the table above, delete keys 2, 7, 13, 15, 29

i. [4 points] Which deletion first causes a reduction in a local depth. **13**

ii. [4 points] Which deletion first causes a reduction in global depth. **13**


## Tree Index

A table index is a replica of a subset of a table's attributes that are organized and/or sorted for efficient access using a subset of those attributes.

The DBMS ensures that the contents of the table and the index are logically in sync.

It is the DBMS's job to figure out the best index(es) to use to execute each query.

There is a trade-off on the number of indexes to create per database.

-> Storage Overhead

-> Maintenance Overhead

### B+Tree

A B+Tree is self-balancing tree data structure that keeps data sorted and allows searches, sequential access, insertions, and deletions in $O(log(n))$

-> Generalization of a binary search tree in that a node can have more than two children.

-> Optimizd for systems that read and write large blocks of data.

#### B+Tree Nodes

Every B+Tree node is comprised of an array of key/value pairs.

-> The keys are derived from the attributes that the index is based on.

-> The values will differ based on whether the node is classified as inner nodes or leaf nodes.

#### B+Tree Leaf Node

![image-20201109142046307](/images/image-20201109142046307.png)

#### Leaf Node values

Approach 1: Record ids

-> A pointer to the location of the tuple that the index entry corresponds to.

Approach 2: Tuple Data

-> The actual contents of the tuple is stored in the leaf node.

-> Secondary indexes have to store the record id as their values.

### B-Tree VS. B+Tree

The original B-Tree stored keys + values in all nodes in the tree

A B+Tree only stores values in leaf nodes. Inner nodes only guide the search process.

### B+Tree Insert

Find correct leaf node L

Put data entry into L in sorted order

If L has enough space, done!

Otherwise, split L keys into L and a new node L2

-> Redistrubute entries evenly, copy up middle keys.

-> Insert index entry pointing to L2 into parent of L.

To split inner node, redistribute entries evenly,but push up middle key.

### B+Tree Delete

Start at root, find leaf L where entry belongs.

Remove the entry.

If L is at least half-full, done!
If L has only M/2-1 entries.

-> Try to re-distribute, borrowing from sibling

-> If re-distribution fails, merge L and sibling

If merge occurred, must delete entry(Pointing to L or sibling) from parent of L.

### Clustered indexes

The table is stored in the sort order specified by the primary key.

-> Can be either heap- or index-organized storage.

Some DBMSs always use a clustered index.

-> If a table doesn't contain a primary key, the DBMS will automatically make a hidden row id primary key.

### Selection condition

The DBMS can use a B+Tree index if the query provides any of the attributes of the search key.

Exaple: Index on **<a, b, c>**

-> Supported: (a = 5 And b = 3)

-> Supported: (b = 3)

Not all DBMSs support this.

For hash index, we must have all attributes in search key.

### Merge Threshold

Some DBMSs do not always merge nodes when it is half full.

Delaying a merge operation may reduce the amount of reorganization.

It may also be better to just let underflows to exist and then periodically rebuild entire tree.

### Variable length keys

Approach #1: Pointers

-> Store the keys as pointers to the tuple’s attribute

Approach #2: Variable Length Nodes

->The size of each node in the index can vary. → Requires careful memory management.

Approach #3: Padding 

->Always pad the key to be max length of the key type.

Approach #4: Key Map / Indirection 

->Embed an array of pointers that map to the key + value list within the node.

![image-20201109143657876](/images/image-20201109143657876.png)

![image-20201109143717331](/images/image-20201109143717331.png)

### INTRA-NODE SEARCH

Approach #1: Linear 

-> Scan node keys from beginning to end.

Approach #2: Binary 

-> Jump to middle key, pivot left/right depending on comparison

### Optimization

#### Prefix compression

Sorted keys in the same leaf node are likely to have the same prefix

![image-20201109144044749](/images/image-20201109144044749.png)

Instead of storing the entire key each time, extract common prefix and store only unique suffix for each key.

#### Suffix truncation

The keys in the inner nodes are only used to "direct traffic"

-> We don't need the entire key.

Store a minimum prefix that is needed to correctly route probes into the index.

![image-20201109144411388](/images/image-20201109144411388.png)

![image-20201109144420908](/images/image-20201109144420908.png)

#### Bulk insert

The fastest/best way to build a B+Tree is to first sort the keys and then build the index from the bottom up.

#### POINTER SWIZZLING

Nodes use page ids to reference other nodes in the index. The DBMS must get the memory location from the page table during traversal.

If a page is pinned in the buffer pool, then we can store raw pointers instead of page ids. This avoids address lookups from the page table.

![image-20201109144558198](/images/image-20201109144558198.png)

![image-20201109144611665](/images/image-20201109144611665.png)

### B+Tree: Duplicate keys

Approach 1: Append Record id

-> Add the tuple's unique record id as part of the key to ensure that all keys are unique.

Approach 2: Overflow leaf nodes

-> Allow leaf nodes to spill into overflow nodes that contain the duplicate keys.

-> This is more complex to maintain and modify.

![image-20201109144810770](/images/image-20201109144810770.png)

![image-20201109144944604](/images/image-20201109144944604.png)

![image-20201109144957932](/images/image-20201109144957932.png)

![image-20201109145012094](/images/image-20201109145012094.png)

![image-20201109145025866](/images/image-20201109145025866.png)

### Implicit indexes

Most DBMSs automatically create an index to enforce integrity constraints but not referential constraints (foreign keys).

-> Primary keys

-> Unique Constrains

![image-20201109145153069](/images/image-20201109145153069.png)

### Partial indexes

Create an index on a subset of the entire table. This potentially reduces its size and the amount of overhead to maintain it.

![image-20201109145305171](/images/image-20201109145305171.png)

One common use case is to partition indexes by date ranges. 

-> Create a separate index per month, year

![image-20201109145409882](/images/image-20201109145409882.png)

### Covering indexes

If all the fields needed to process the query are available in an index, then the DBMS does not need to retrieve the tuple

![image-20201109145500432](/images/image-20201109145500432.png)

### Index include columns

Embed additional columns in indexes to support index-only queries.

![image-20201109145610799](/images/image-20201109145610799.png)

### Trie index

Use a digital representation of keys to examine prefixes oneby-one instead of comparing entire key

![image-20201109145754510](/images/image-20201109145754510.png)

Shape only depends on key space and lengths.

-> Does not depend on existing keys or insertion order.

-> Does not require rebalancing operations.

All operations have O(k) complexity where k is the length of the key.


## Index Concurrency 

### L ATCH IMPLEMENTATIONS

Approach #1: Blocking OS Mutex

-> Simple to use

-> Non-scalable (about 25ns per lock/unlock invocation)

-> Example: std::mutex

Approach #2: Test-and-Set Spin Latch (TAS)

-> Very efficient (single instruction to latch/unlatch) 

-> Non-scalable, not cache friendly 

-> Example: std::atomic

Approach #3: Reader-Writer Latch

-> Allows for concurrent readers 

-> Must manage read/write queues to avoid starvation 

-> Can be implemented on top of spinlocks

### HASH TABLE L ATCHING

Easy to support concurrent access due to the limited ways threads access the data structure.

-> All threads move in the same direction and only access a single page/slot at a time. 

-> Deadlocks are not possible.

#### Approach #1: Page Latches

Each page has its own reader-write latch that protects its entire contents

Threads acquire either a read or write latch before they access a page.

#### Approach #2: Slot Latches

Each slot has its own latch

Can use a single mode latch to reduce meta-data and computational overhead.

### B+TREE CONCURRENCY CONTROL

We want to allow multiple threads to read and update a B+Tree at the same time.

We need to protect from two types of problems:

→ Threads trying to modify the contents of a node at the same time.

→ One thread traversing the tree while another thread splits/merges nodes.

![image-20201109150931585](/images/image-20201109150931585.png)

![image-20201109150954823](/images/image-20201109150954823.png)

![image-20201109151008219](/images/image-20201109151008219.png)

![image-20201109151022009](/images/image-20201109151022009.png)

![image-20201109151039941](/images/image-20201109151039941.png)

### L ATCH CRABBING/COUPLING

Protocol to allow multiple threads to access/modify B+Tree at the same time.

Basic Idea:

→ Get latch for parent.

→ Get latch for child

→ Release latch for parent if “safe”.

A safe node is one that will not split or merge when updated.

→ Not full (on insertion)

→ More than half-full (on deletion)

Find: Start at root and go down; repeatedly,

→ Acquire R latch on child

→ Then unlatch parent

Insert/Delete: Start at root and go down

obtaining W latches as needed. Once child is latched, check if it is safe:

→ If child is safe, release all latches on ancestors.

![image-20201109151241125](/images/image-20201109151241125.png)

![image-20201109151252298](/images/image-20201109151252298.png)

![image-20201109151306328](/images/image-20201109151306328.png)

![image-20201109151323289](/images/image-20201109151323289.png)

![image-20201109151334621](/images/image-20201109151334621.png)

![image-20201109151346072](/images/image-20201109151346072.png)

![image-20201109151357950](/images/image-20201109151357950.png)

What was the first step that all the update examples did on the B+Tree? 

Taking a write latch on the root every time becomes a bottleneck with higher concurrency. 

Can we do better?

Assume that the leaf node is safe. 

Use read latches and crabbing to reach it, and then verify that it is safe. 

If leaf is not safe, then do previous algorithm using write latches.

### BET TER L ATCHING ALGORITHM

Search: Same as before.

Insert/Delete: 

→ Set latches as if for search, get to leaf, and set W latch on leaf.

→ If leaf is not safe, release all latches, and restart thread using previous insert/delete protocol with write latches.

This approach optimistically assumes that only leaf node will be modified; if not, R latches set on the first pass to leaf are wasteful.

The threads in all the examples so far have acquired latches in a "top-down" manner.

→ A thread can only acquire a latch from a node that is below its current node.

→ If the desired latch is unavailable, the thread must wait until it becomes available.

But what if we want to move from one leaf node to another leaf node?

### LEAF NODE SCANS

Latches do not support deadlock detection or avoidance. The only way we can deal with this problem is through coding discipline.

The leaf node sibling latch acquisition protocol must support a "no-wait" mode.

The DBMS's data structures must cope with failed latch acquisitions
