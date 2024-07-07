# Chapter 3. Storage and Retrieval

## Data Structures That Power Your Database

- Many databases internally use a _log_, an **append-only** data file, which has pretty good performance and generally very efficient.
- Real databases have more issues to deal with (such as **concurrency** control, **reclaiming disk** space so that the log doesn’t grow forever, handling **errors**, **partially written** records, and so on) but the basic principle is the same. Logs are incredibly useful.
- In order to efficiently find the value for a particular key in the database, we need a different data structure: an **index**.
  - The idea behind them is simple: keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want.
  - If you want to search the same data in several different ways, you may need several different indexes on different parts of the data.
- An index is an additional structure that is derived from the primary data—many databases allow you to add and remove indexes, and this doesn’t affect the contents of the database, it only affects the performance of queries. Maintaining additional structures is **overhead**, especially on **writes**. For writes, it’s hard to beat the performance of simply appending to a file, so any kind of index usually slows down writes.
- This is an important trade-off in storage systems: well-chosen indexes can **speed up read** queries, but every index **slows down writes**. For this reason, databases don’t usually index everything **by default**, but require you to choose indexes manually, using your knowledge of the application’s typical query patterns.

### Hash indexes

- Key-value stores are quite similar to the dictionary type that you can find in most programming languages, and which is usually implemented as a **hash map** (hash table).
- Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value. <p align="center"><img src="assets/hash-indexes.png"></p>
- How do we avoid eventually running out of disk space? A good solution is to break the log into **segments** of a certain size, and to periodically run a background process for **compaction and merging** of segments.
- **Compaction** means throwing away all key-value pairs in the log except for the most recent update for each key. This makes the segments **much smaller** (assuming that every key is updated multiple times on average), so we can also **merge** several segments into one.
- Each segment now has its **own in-memory** hash table, mapping keys to file offsets. In order to find the value for a key, we first check the **most recent** segment’s hash map; if the key is not present we check the second-most-recent segment, and so on.
- Pros :+1::
  - Appending and segment merging are **sequential write operations**, which are generally much faster than **random writes**. This performance difference applies both to traditional **spinning-disk** hard drives and to **flash-based** solid state drives (SSDs).
  - **Concurrency** and **crash recovery** are much simpler if files are immutable. For example, you don’t have to worry about the case where a crash happened while a value was being overwritten, leaving you with a file containing part of the old and part of the new value spliced together.
  - Merging old segments avoids problems of data files getting **fragmented** over time.
- Cons :-1::
  - The hash table must **fit in memory**, so if you have a very large number of keys, you’re out of luck. In principle, you could maintain a hash map on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O, it is expensive to grow when it becomes full, and hash collisions require fiddly logic.
  - **Range queries** are not efficient. For example, you cannot easily fetch the values for all keys between `kitty00000` and `kitty99999`, you’d have to look up each key individually in the hash maps.
- 👁️ Example of storage engines using a log-structured hash table: [Bitcask](https://en.wikipedia.org/wiki/Bitcask).

## SSTables and LSM-trees

- In **Sorted String Table**, or SSTable for short, we require that the sequence of key-value pairs is **sorted by key**. We also require that each key only **appears once** within each merged segment file (the compaction process already ensures that).
- SSTables have several big advantages over log segments with hash indexes 👍:
  - **Merging segments is simple and efficient**, even if the files are bigger than the available memory. You start reading the input files side-by-side, look at the first key in each file, copy the lowest-numbered key to the output file, and repeat. If the same key appears in several input files, keep the one from the most recent input file, and discard the values in older segments. This produces a new merged segment which is also sorted by key, and which also has exactly one value per key.
  - In order to find a particular key in the file, you **no longer need to keep an index** of all the keys in memory. You still need an in-memory index to tell you the **offsets for some of the keys**, but it can be sparse: one key for every few kilobytes of segment file is sufficient, because a few kilobytes can be scanned very quickly.
  - Since read requests need to scan over several key-value pairs in the requested range anyway, it is possible to **group those records into a block and compress it** before writing it to disk. Each entry of the sparse in-memory index then points at the start of a compressed block. Nowadays, **disk bandwidth is usually a worse bottleneck than CPU**, so it is worth spending a few additional CPU cycles to reduce the amount of data you need to write to and read from disk.

### Constructing and maintaining SSTables

- Maintaining a sorted structure on disk is possible, but maintaining it in memory is much easier. There are plenty of well-known tree data structures that you can use, such as _Red-Black trees_ or _AVL_ trees. With these data structures, you can insert keys in any order, and read them back in **sorted** order.
- When a write comes in, add it to an in-memory balanced tree data structure. This in-memory tree is sometimes called a **memtable**.
- When the memtable gets bigger than some threshold — typically a few megabytes — write it out to disk as an **SSTable** file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key. The new SSTable file becomes the most recent segment of the database. When the new SSTable is ready, the memtable can be emptied.
- The basic idea - keeping a cascade of SSTables that are merged in the **background** - is simple and effective. Even when the dataset is much bigger than memory it continues to work well. Since data is stored in sorted order, you can efficiently perform range queries (scanning all keys above some minimum and up to some maximum). And because the disk writes are sequential, the LSM-tree can support remarkably **high write throughput**.

### Making an LSM-tree out of SSTables

- ⭐The algorithm described here is essentially what is used in **LevelDB** and **RocksDB**, key-value storage engine libraries.
- Originally this indexing structure was described by *Patrick O’Neil* under the name **Log-Structured Merge-Tree** building on earlier work on log-structured filesystems. Storage engines that are based on this principle of merging and compacting sorted files are often called **LSM storage engines**
- ⭐ **Lucene**, an indexing engine for FTS used by **Elasticsearch** and **Solr**, uses a similar method for storing its term dictionary. A full-text index is much more
complex than a KV index but is based on a similar idea:
  - Given a word in a search query, find all the documents that mention the word. This is implemented with a KV structure where the key is a word (a **term**) and the value is the list of IDs of all the documents that contain the word (the **postings list**). In Lucene, this mapping from term to postings list is kept in SSTable-like sorted files, which are merged in the background as needed.

### Performance optimizations

- LSM-tree algorithm can be **slow** when looking up keys that do **not exist** in the database:
  1. You have to check the memtable,
  2. Then the segments all the way back to the **oldest** (possibly having to read from disk for each one) before you can be sure that the key does not exist.
- In order to optimize this kind of access, storage engines often use additional **Bloom filters**.
- There are also different strategies to determine the **order** and **timing** of how SSTables are compacted and merged. The most common options are **size-tiered** and **leveled compaction**.
  - ▶️ **LevelDB** and **RocksDB** use leveled compaction (hence the name of Lev‐lDB), **HBase** uses size-tiered, and **Cassandra** supports both.

## B-trees

- Are the most widely-used indexing structure.
- Remain the standard index implementation in almost all relational databases, and many non-relational databases use them too.
- The **log-structured indexes** we saw earlier break the database down into **variable-size segments**, typically several megabytes or more in size, and always write a segment sequentially. By contrast, **B-trees** break the database down into **fixed-size blocks** or pages, traditionally `4 kB` in size, and read or write one page at a time. This corresponds more closely to the underlying hardware, as disks are also arranged in fixed-size blocks.
- Each page can be identified using an address or location, which allows one page to refer to another—similar to a **pointer**, but on disk instead of in memory. We can use these page references to construct a tree of pages: <p align="center"><img src="assets/b-trees-structure.png"></p>
- The number of references to child pages in one page of the B-tree is called the **branching factor**.
- A four-level tree of 4 KB pages with a branching factor of 500 can store up to 256 TB 🆗.

### Making B-trees reliable

- Some operations require several **different pages** to be overwritten. For example, if you split a page because an insertion caused it to be overfull, you need to write the two pages that were split, and also overwrite their parent page to update the references to the two child pages.
  - ⚠️ This is a dangerous operation, because if the DB crashes after only some of the pages have been written, you end up with a corrupted index.
  - In order to make the DB resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a **write-ahead log** (WAL, also known as a *redo log*). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself. When the DB comes back up after a crash, this log is used to restore the B-tree back to a consistent state.
- An additional complication of updating pages in place is that careful **concurrency** control is required if multiple threads are going to access the B-tree at the same time - otherwise a thread may see the tree in an inconsistent state.
  - This is typically done by protecting the tree’s data structures with **latches** (lightweight locks). Log-structured approaches are simpler in this regard, because they do all the merging in the background **without interfering** with incoming queries and atomically swap old segments for new segments from time to time.

### B-tree optimizations

- Instead of overwriting pages and maintaining a WAL for crash recovery, some databases (like **LMDB**) use a CoW scheme.
- We can save space in pages by not storing the entire key, but **abbreviating** it. keys only need to provide enough information to act as boundaries between key ranges.
  - ▶️ Packing more keys into a page allows the tree to have a **higher branching factor**, and thus **fewer levels**.

### Comparing B-Trees and LSM-Trees

- As a rule of thumb, LSM-trees are typically faster for **writes**, whereas B-trees are thought to be faster for **reads**.
- Reads are typically slower on LSM-trees because they have to check several different data structures and SSTables at different stages of compaction.
- 👍 Advantages of LSM-trees:
  - A B-tree index must write every piece of data at least **twice**: once to the write-ahead log, and once to the tree page itself (and perhaps again as pages are split).
  - Log-structured indexes also rewrite data **multiple** times due to repeated compaction and merging of SSTables. This effect —one write to the database resulting in multiple writes to the disk over the course of the database’s lifetime— is known as **write amplification**.
  - LSM-trees are typically able to sustain **higher write throughput** than Btrees, partly because they sometimes have **lower write amplification** (although this depends on the storage engine configuration and workload), and partly because they **sequentially** write compact SSTable files rather than having to overwrite several pages in the tree. This difference is particularly important on magnetic hard drives, where sequential writes are much faster than random writes.
  - LSM-trees can be **compressed** better, and thus often produce smaller files on disk than B-trees.


## Multi-column indexes

- The most common type of multi-column index is called a **concatenated index**, which simply combines several fields into one key by appending one column to another (the index definition specifies in which order the fields are concatenated).
- Multi-dimensional indexes are a more general way of querying several columns at once, which is particularly important for **geospatial** data:

```sql
SELECT * FROM restaurants
WHERE latitude > 51.4946 AND latitude < 51.5079 AND longitude > -0.1162 AND longitude < -0.1004;
```

- A standard B-tree or LSM-tree index is not able to answer that kind of query efficiently: it can give you either all the restaurants in a range of latitudes (but at any longitude), or all the restaurants in a range of longitudes (but anywhere between north and south pole), but not both simultaneously.
- One option is to translate a two-dimensional location into a single number using a space-filling curve, and then to use a regular B-tree index. More commonly, specialized spatial indexes such as R-trees are used. For example, PostGIS implements geospatial indexes as R-trees using PostgreSQL’s Generalized Search Tree indexing facility.
