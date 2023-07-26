# Chapter 3. Storage and Retrieval

- Many databases internally use a _log_, an **append-only** data file, which has pretty good performance and generally very efficient.
- Real databases have more issues to deal with (such as concurrency control, reclaiming disk space so that the log doesn’t grow forever, handling errors, partially written records, and so on) but the basic principle is the same. Logs are incredibly useful.
- In order to efficiently find the value for a particular key in the database, we need a different data structure: an **index**. The idea behind them is simple: keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want. If you want to search the same data in several different ways, you may need several different indexes on different parts of the data.
- An index is an additional structure that is derived from the primary data—many databases allow you to add and remove indexes, and this doesn’t affect the contents of the database, it only affects the performance of queries. Maintaining additional structures is **overhead**, especially on **writes**. For writes, it’s hard to beat the performance of simply appending to a file, so any kind of index usually slows down writes.
- This is an important trade-off in storage systems: well-chosen indexes can speed up read queries, but every index slows down writes. For this reason, databases don’t usually index everything **by default**, but require you—the application developer or database administrator—to choose indexes manually, using your knowledge of the application’s typical query patterns.

## Hash indexes

- Key-value stores are quite similar to the dictionary type that you can find in most programming languages, and which is usually implemented as a **hash map** (hash table).
- Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value. <p align="center"><img src="assets/hash-indexes.png"></p>
- How do we avoid eventually running out of disk space? A good solution is to break the log into **segments** of a certain size, and to periodically run a background process for **compaction and merging** of segments, illustrated in Figure 3-2.
  - **Compaction** means throwing away all key-value pairs in the log except for the most recent update for each key. This makes the segments **much smaller** (assuming that every key is updated multiple times on average), so we can also merge several segments into one.
- Pros :+1::
  - Appending and segment merging are **sequential write operations**, which are generally much faster than **random writes**. This performance difference applies both to traditional spinning-disk hard drives and to flash-based solid state drives (SSDs).
  - **Concurrency** and **crash recovery** are much simpler if files are immutable. For example, you don’t have to worry about the case where a crash happened while a value was being overwritten, leaving you with a file containing part of the old and part of the new value spliced together.
  - Merging old segments avoids problems of data files getting **fragmented** over time.
- Cons :-1::
  - The hash table must **fit in memory**, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O, it is expensive to grow when it becomes full, and hash collisions require fiddly logic.
  - **Range queries** are not efficient. For example, you cannot easily fetch the values for all keys between `kitty00000` and `kitty99999`, you’d have to look up each key individually in the hash maps.

## SSTables and LSM-trees

- In **Sorted String Table**, or SSTable for short, we require that the sequence of key-value pairs is **sorted by key**. We also require that each key only **appears once** within each merged segment file (the merging process already ensures that).
- Pros :+1::
  - **Merging segments is simple and efficient**, even if the files are bigger than the available memory. You start reading the input files side-by-side, look at the first key in each file, copy the lowest-numbered key to the output file, and repeat. If the same key appears in several input files, keep the one from the most recent input file, and discard the values in older segments. This produces a new merged segment which is also sorted by key, and which also has exactly one value per key.
  - In order to find a particular key in the file, you **no longer need to keep an index** of all the keys in memory. You still need an in-memory index to tell you the offsets for some of the keys, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly.
  - Since read requests need to scan over several key-value pairs in the requested range anyway, it is possible to **group those records into a block and compress it** before writing it to disk. Each entry of the sparse in-memory index then points at the start of a compressed block. Nowadays, **disk bandwidth is usually a worse bottleneck than CPU**, so it is worth spending a few additional CPU cycles to reduce the amount of data you need to write to and read from disk.
- Maintaining a sorted structure on disk is possible, but maintaining it in memory is much easier. There are plenty of well-known tree data structures that you can use, such as _Red-Black trees_ or _AVL_ trees. With these data structures, you can insert keys in any order, and read them back in sorted order.
- When a write comes in, add it to an in-memory balanced tree data structure. This in-memory tree is sometimes called a **memtable**.
- When the memtable gets bigger than some threshold—typically a few megabytes—write it out to disk as an SSTable file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key. The new SSTable file becomes the most recent segment of the database. When the new SSTable is ready, the memtable can be emptied.
- The basic idea—keeping a cascade of SSTables that are merged in the background—is simple and effective. Even when the dataset is much bigger than memory it continues to work well. Since data is stored in sorted order, you can efficiently perform range queries (scanning all keys above some minimum and up to some maximum). And because the disk writes are sequential, the LSM-tree can support remarkably **high write throughput**.

## B-trees

- Are the most widely-used indexing structure.
- Remain the standard index implementation in almost all relational databases, and many non-relational databases use them too.
- The **log-structured indexes** we saw earlier break the database down into **variable-size segments**, typically several megabytes or more in size, and always write a segment sequentially. By contrast, **B-trees** break the database down into **fixed-size blocks** or pages, traditionally `4 kB` in size, and read or write one page at a time. This corresponds more closely to the underlying hardware, as
  disks are also arranged in fixed-size blocks.

## Multi-column indexes

- The most common type of multi-column index is called a **concatenated index**, which simply combines several fields into one key by appending one column to another (the index definition specifies in which order the fields are concatenated).
- Multi-dimensional indexes are a more general way of querying several columns at once, which is particularly important for **geospatial** data:

```sql
SELECT * FROM restaurants
WHERE latitude > 51.4946 AND latitude < 51.5079 AND longitude > -0.1162 AND longitude < -0.1004;
```

- A standard B-tree or LSM-tree index is not able to answer that kind of query efficiently: it can give you either all the restaurants in a range of latitudes (but at any longitude), or all the restaurants in a range of longitudes (but anywhere between north and south pole), but not both simultaneously.
- One option is to translate a two-dimensional location into a single number using a space-filling curve, and then to use a regular B-tree index. More commonly, specialized spatial indexes such as R-trees are used. For example, PostGIS implements geospatial indexes as R-trees using PostgreSQL’s Generalized Search Tree indexing facility.
