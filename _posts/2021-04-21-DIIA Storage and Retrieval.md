# DIIA Storage and Retrieval

主要探讨两种类型的storage engine: **log-structured** storage engines and **page-oriented** storage engine

## Data Structures That Power You Database

首先思考一下，最简单的数据存储方式是什么？

我们可以实现两个简单的BASH函数

```sh
#!/bin/bash
db_set () {
	echo "$1, $2" >> database
}

db_get () {
	// 首先用grep在database中搜索key对应的所有项，然后用sed将"$1,"这个部分替换成空，只留下
	// value值, 然后用tail取最新值。
	grep "^$1, " database | sed -e "s/^$1,//" | tail -n 1
}
```

用这两个函数就可以实现一个简单的kv store，一个文本文件中的每行都存储一个键值对，键值对使用逗号作为间隔符。db_set操作将键值对append到文本文件的末尾，所以新插入的值总是在末尾。

类似的，一些数据库使用log作为存储方式，log就是**an append-only sequence of records**, 但是数据库还有其他的问题需要解决，例如并发控制，回收磁盘空间，错误处理(防止出现partially written records)

我们可以发现使用db_get的读性能非常差，需要遍历所有的records，比较key值是否相等，从算法的角度来说，时间复杂度为$O(n)$。为了能够高效的检索数据，我们引入**Index**。

Index是用来加快查询效率的一种数据结构，里面存放的一般是元数据，通常我们可以根据需要增加或者删除索引。增加索引会加速读，但是会增加写操作的overhead，因为每次对数据进行修改，我们也要想应的修改index中的对应项。**Index必须与primary data保持同步**。

简单介绍一下Bitcast(default storage engine in Riak)，因为hash map完全存放在内存中，所以Bitcask提供高性能的读写操作，但是要求所有的key都能装入内存。values不要求能够完全装入内存，因为根据hash map，我们可以根据key快速的找到value的字节地址，通过一次seek操作将对应value从disk读入到内存中。如果data file的一部分已经缓存在filesystem cache中，那么read操作不需要执行任何disk I/O。

像Bitcask这样的存储引擎适合在键对应的值需要经常修改的情况下使用。

Log-Structured storage engine append操作非常快。如果所有的key能够装入到内存中，那么我们可以在内存中建立Hash index，index项的index_key为key， index_value为value在disk中的地址。这样也可以实现快速的读操作。

通常我们会将log分成多个Segment，对log的写入是写入到对应的Segment中，当Segment达到一定的大小，关闭当前写入的Segment，创建一个新的Segment继续进行写入。之后，系统可以对已经关闭的Segments进行Compaction操作。

Compaction操作就是：删除所有的重复的键值对，只保留最佳写入的键值对。例如:

```
insert(1, 2)
insert(1, 3)
```

那么在Compaction时，{1:2}这个记录会被删除。

同时因为Compaction操作会使得Segment大小变小，所以我们还需要(**Merge**)合并多个小的Segment。由于Segment在写入后不能对写入内容进行任何的修改，所以Merge操作是创建一个新的Segment，将多个Segment的内容写入到新Segment中。且由于Segment的immutable特性，在系统进行Compaction && Merge操作时，可以使用Old segments进行读、写操作。

**[注] Log-Structured Log 只能进行Append操作，不能进行修改和删除**

当Compaction && Merge操作完成时，我们可以将read request reroute到合并后的Segment上，然后Old Segments可以被安全的删除。

所以现在每个Segment都有自己的in-memory Hash index。那么对于一个Read操作，我们首先在最新的Segment的Hash Index上检索，如果没有，在第二新的Segment的Hash Index上进行检索，以此类推。

### Issues

1. Delete: 删除操作不会直接在对于位置进行修改，而是Append一个tombstore record表示这个key被删除了。当系统进行Merge操作时，Merge进程遇到了一个tombstone record, 那么进程会记住对于的key，然后Merge Older records时，如果碰到对于的key，直接放弃对于的键值对。
2. Crash recovery: 由于Hash Index存放在内存中，所以Crash会导致内存中的Index丢失。所以可以使用WAL，即在写入Index之前，先写入log，相当于对Hash index建立快照。
3. Partially Written records: 如果在写入Record的过程中crash了，那么对于的record一半时更新的值，一半时旧值。Bitcask使用checksum，在每个record后面增加一个checksum值，那么read操作的时候，计算对于record的checksum值与末尾的checksum值比较，如果不同说明该record is corrupted。
4. Concurrency control: 理论上来说Append操作是原子的。

所以为什么不直接修改旧值呢，而是进行Append-only操作呢？

1. Append操作和Segment merge操作都是sequential write操作，速度像比与Random Write要快很多。
2. 如果log是Append-only and Immutable，那么并发控制和crash recovery会更简单
3. Merge操作避免使得Segment文件出现很多碎片。

但同时Hash table index也会带来一些局限性：

1. Hash table必须能够装入内存。
2. Range query很慢，显然Hash index不适用于Range query

## SSTables and LSM-Trees

SSTable就是Sorted String Table。这要求每个Segment中的键值对按照key值排序。同时要求在每个merged segment文件中，每个key值只出现一次，且是最新更新的那个键值对。(Compaction操作就是完成这个任务)。与之前的基于Hash index的Log file相比，SSTables有以下几个优势：

1. 合并操作变得更简单，即使文件大于可用内存。SSTables的合并操作类似于归并排序。如果在输入的Segments中出现了相同的值，那么我们只保留最近写入的Segment的对于键值对。
2. 由于Segment现在的键值对有序，我们现在可以使用Sparse Index，也就是说对于每个Segment，我们需要对每个key建立Index，我们可以按照几Kb的间隔建立Index。那么要检索一个key的时候，我们只需要找到这个key会出现在那个区间，然后区间中顺序搜索(二分查找)就行了。

由于write的写入是任意顺序的，我们如何保证Segment里的键值对有序呢？

我们可以在内存中维护一个平衡二叉树的数据结构(比如红黑树或者AVL树)，这个In-memory tree被称为MemTable。所以我们进行写操作时，将对应的键值对写到MemTable中。当MemTable达到指定的大小的时候，我们将MemTable中的所有数据写入到Segment中，此时Segment中的键值对是有序的。当MemTable写入Segment时，我们可以创建一个新的MemTable用于处理写入操作。

为了处理读操作，我们首先检查Memtable，然后检索最新的Segment的index，之后依次检查次新的Segments的index。

为了避免Crash时，内存中的MemTable中的数据丢失，使用WAL。

**使用类似算法的Storage Engine包括： LevelDB, RocksDB, Cassandra, HBase**

**这类merging and compacting sorted files storage engine通常被称为LSM Storage Engine。**

## 性能优化

上述的LSM-tree算法存在一个问题：当检索的key不存在的时候，我们需要检索MemTable和所有的Segments，这意味着我们要访问#Segments次disk。我们可以使用bloom filter。
