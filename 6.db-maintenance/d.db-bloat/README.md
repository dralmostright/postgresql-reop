### Table/Index Bloat

Bloating in database is created when tables or indexes are updated, an update is essentially a delete and insert operation. The diskspace used by the delete is available for reuse but it is not reclaimed hence creating the bloat. Same is the case with PostgreSQL database, frequent UPDATE and DELETE operations can leave a lot of unused space in table or index relation files on disk. Over the time this space can build up and cause the performance degradation for both tables and indexes. This buildup is referred to as bloated tables or indexes.

#### How does it happen:
The PostgreSQL system implements the MVCC (Multiversion Concurrency Control) and the way MVCC is implemented, it can introduce bloat’s in your system.

When an UPDATE or DELETE statement is used in PostgreSQL, it does not physically remove that row from the disk. In an UPDATE case, it marks the effected rows as invisible and INSERTs the new version of those rows. While DELETE is little simple the effected rows are just marked as invisibles. These invisibles rows are also called dead rows or dead tuples.

What this means? and why is it of any significance? Over the time these dead tuples can accumulate to a huge number and in some worst case scenarios, it possible that this accumulation is even greater that the actual rows in the table becomes unusable. 

You see, these rows were marked invisible but they are still part of the table and are consuming the disk space… Assuming there are a million rows in a table, where each row takes 100bytes of disk space. This table is assumed to be consuming around 100MB of disk space. Now let’s assume there are 30% invisible rows present in the table, that would mean that 130MB’s of disk space is being utilised by the dead tuples. This looks insignificant amount but consider the real world scenario where tables use GB/TB’s of data and it becomes a serious problem.

**** Why Is Table Bloat an Issue? 

There are several reasons why table bloat should be on your radar if you’re scaling PostgreSQL or running large transactional PostgreSQL tables:

* Table bloat causes performance degradation. Dead tuples get stored alongside live tuples. During query execution, PostgreSQL has to sift through more data blocks, many of which are filled with dead tuples, causing increased  I/O and slower queries.  

* Indexes also become bloated, leading to even slower lookup times. Indexes, at their core, are data structures that provide a fast path to rows in a table based on the values of one or several columns. When rows are updated or deleted in a table, the associated indexed values are not immediately purged from the index. Instead, similar to the MVCC mechanism for tables, these outdated index entries are marked as invalid but are not immediately removed. Over time, as more rows are modified and deleted, these "dead" index entries accumulate and contribute to index bloat.

* Wasted disk space. Dead tuples and excessive page allocations also cause your disk usage to be higher than needed, meaning more infrastructure costs. 

* Operational challenges. Everything slows down if your table is bloated, as it’s essentially larger—e.g., if you're using replication, bloated tables can cause replication lag since the replication process needs to handle more data than necessary; backing up a bloated table takes longer and requires more storage; restoring from a backup can become more time-consuming; and so on. 
