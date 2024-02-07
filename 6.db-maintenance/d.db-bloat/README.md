### Table/Index Bloat

Bloating in database is created when tables or indexes are updated, an update is essentially a delete and insert operation. The diskspace used by the delete is available for reuse but it is not reclaimed hence creating the bloat. Same is the case with PostgreSQL database, frequent UPDATE and DELETE operations can leave a lot of unused space in table or index relation files on disk. Over the time this space can build up and cause the performance degradation for both tables and indexes. This buildup is referred to as bloated tables or indexes.

#### How does it happen:
The PostgreSQL system implements the MVCC (Multiversion Concurrency Control) and the way MVCC is implemented, it can introduce bloat’s in your system.

When an UPDATE or DELETE statement is used in PostgreSQL, it does not physically remove that row from the disk. In an UPDATE case, it marks the effected rows as invisible and INSERTs the new version of those rows. While DELETE is little simple the effected rows are just marked as invisibles. These invisibles rows are also called dead rows or dead tuples.

What this means? and why is it of any significance? Over the time these dead tuples can accumulate to a huge number and in some worst case scenarios, it possible that this accumulation is even greater that the actual rows in the table becomes unusable. 

You see, these rows were marked invisible but they are still part of the table and are consuming the disk space… Assuming there are a million rows in a table, where each row takes 100bytes of disk space. This table is assumed to be consuming around 100MB of disk space. Now let’s assume there are 30% invisible rows present in the table, that would mean that 130MB’s of disk space is being utilised by the dead tuples. This looks insignificant amount but consider the real world scenario where tables use GB/TB’s of data and it becomes a serious problem.

#### Why Is Table Bloat an Issue? 

There are several reasons why table bloat should be on your radar if you’re scaling PostgreSQL or running large transactional PostgreSQL tables:

* Table bloat causes performance degradation. Dead tuples get stored alongside live tuples. During query execution, PostgreSQL has to sift through more data blocks, many of which are filled with dead tuples, causing increased  I/O and slower queries.  

* Indexes also become bloated, leading to even slower lookup times. Indexes, at their core, are data structures that provide a fast path to rows in a table based on the values of one or several columns. When rows are updated or deleted in a table, the associated indexed values are not immediately purged from the index. Instead, similar to the MVCC mechanism for tables, these outdated index entries are marked as invalid but are not immediately removed. Over time, as more rows are modified and deleted, these "dead" index entries accumulate and contribute to index bloat.

* Wasted disk space. Dead tuples and excessive page allocations also cause your disk usage to be higher than needed, meaning more infrastructure costs. 

* Operational challenges. Everything slows down if your table is bloated, as it’s essentially larger—e.g., if you're using replication, bloated tables can cause replication lag since the replication process needs to handle more data than necessary; backing up a bloated table takes longer and requires more storage; restoring from a backup can become more time-consuming; and so on. 

#### What Increases Table Bloat? 
As we said previously, table bloat is, unfortunately, a natural consequence of how PostgreSQL manages data. But it is also true that certain workload patterns lead to more bloat than others, for example: 

* High volume of UPDATEs/DELETEs. Tables with frequent updates can quickly accumulate these dead rows since when a row is updated in Postgres, a new version of the row is created, and the old version is marked as "dead". Similarly, when rows are deleted, they aren't immediately removed from the table. Instead, they're marked for deletion and later removed by the autovacuum process (more about autovacuum later).

* Bulk INSERTs followed by DELETEs. Batch operations, like inserting a large number of rows followed by deleting many of them, can lead to rapid bloat accumulation. This is especially true if these operations are frequent and autovacuum doesn't have enough time or resources to clean up between these batches.

* Long-running transactions. Long-running transactions can prevent dead rows from being vacuumed. Until a transaction is complete, the rows that were modified or deleted during that transaction cannot be vacuumed, even if they're no longer needed. This can lead to a temporary buildup of bloat.

#### Decting BLOATS
One of the most popular and efficient tools for this purpose in PostgreSQL is pgstattuple: 

Lets review the extensions and if not available lets create one.
```
[postgres@testdb ~]$ psql
psql (15.5)
Type "help" for help.

postgres=# \c testdb
You are now connected to database "testdb" as user "postgres".
testdb=#
testdb=# \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
(1 row)

testdb=# CREATE EXTENSION pgstattuple;
CREATE EXTENSION
testdb=# \dx
                   List of installed extensions
    Name     | Version |   Schema   |         Description
-------------+---------+------------+------------------------------
 pgstattuple | 1.5     | public     | show tuple-level statistics
 plpgsql     | 1.0     | pg_catalog | PL/pgSQL procedural language
(2 rows)

testdb=# SELECT oid, extname, extversion FROM pg_extension;
  oid  |   extname   | extversion
-------+-------------+------------
 13527 | plpgsql     | 1.0
 24604 | pgstattuple | 1.5
(2 rows)

testdb=#
```
Note: The extension needs to be created on each database.


Now lets create table and index and perform some deletes:
```
testdb=# CREATE TABLE test as SELECT x, md5(random()::text) as y FROM generate_Series(1, 1000000) x;
SELECT 1000000
testdb=# CREATE INDEX ON test (x);
CREATE INDEX
testdb=#
testdb=# SELECT pg_size_pretty(pg_relation_size('test')) as table_size,
testdb-# pg_size_pretty(pg_relation_size('test_x_idx')) as index_size,
testdb-# (pgstattuple('test')).dead_tuple_percent;
 table_size | index_size | dead_tuple_percent
------------+------------+--------------------
 65 MB      | 21 MB      |                  0
(1 row)

testdb=#
testdb=# DELETE FROM test WHERE x % 3 = 0;
DELETE 333333
testdb=# ANALYZE test;
ANALYZE
testdb=# SELECT pg_size_pretty(pg_relation_size('test')) as table_size,
pg_size_pretty(pg_relation_size('test_x_idx')) as index_size,
(pgstattuple('test')).dead_tuple_percent;
 table_size | index_size | dead_tuple_percent
------------+------------+--------------------
 65 MB      | 21 MB      |              29.78
(1 row)

testdb=#
```

See the table size remains the same, however the output of pgstattuple shows that 29.78% of disk space is wasted. It’s taking the space in table but not useable anymore. Now let’s take a look at the index.
```
testdb=# SELECT pg_relation_size('test') as table_size,
testdb-#  pg_relation_size('test_x_idx') as index_size,
testdb-# 100-(pgstatindex('test_x_idx')).avg_leaf_density as bloat_ratio;
 table_size | index_size | bloat_ratio
------------+------------+-------------
   68272128 |   22487040 |       39.86
(1 row)

testdb=#
```
After the above operations, index has become around 40% bloated. That means that the performance of this index will degrade because that much entries are either empty or pointing to dead tuples.

#### Removing Bloats
There are couple of ways which can be utilised to avoid the bloat in tables and indexes.

1. VACUUM

The VACUUM command adds expired rows to the free space map so that the space can be reused. When VACUUM is run regularly on a table that is frequently updated, the space occupied by the expired rows can be promptly reused, preventing the table file from growing larger. It is also important to run VACUUM before the free space map is filled. For heavily updated tables, you may need to run VACUUM at least once a day to prevent the table from becoming bloated.

When a table accumulates significant bloat, running the VACUUM command is insufficient. For small tables, running VACUUM FULL <table_name> can reclaim space used by rows that overflowed the free space map and reduce the size of the table file. However, a VACUUM FULL statement is an expensive operation that requires an ACCESS EXCLUSIVE lock and may take an exceptionally long and unpredictable amount of time to finish for large tables. You should run VACUUM FULL on tables during a time when users and applications do not require access to the tables being vacuumed, such as during a time of low activity, or during a maintenance window.

<strong>Warning</strong>: When a table is significantly bloated, it is better to run VACUUM before running ANALYZE. Analyzing a severely bloated table can generate poor statistics if the sample contains empty pages, so it is good practice to vacuum a bloated table before analyzing it.

```
testdb=# SELECT pg_size_pretty(pg_relation_size('test')) as table_size,
pg_size_pretty(pg_relation_size('test_x_idx')) as index_size,
(pgstattuple('test')).dead_tuple_percent;
 table_size | index_size | dead_tuple_percent
------------+------------+--------------------
 65 MB      | 21 MB      |                  0
(1 row)

testdb=# vacuum verbose analyze test;
INFO:  vacuuming "testdb.public.test"
INFO:  finished vacuuming "testdb.public.test": index scans: 0
pages: 0 removed, 8334 remain, 1 scanned (0.01% of total)
tuples: 0 removed, 666667 remain, 0 are dead but not yet removable
removable cutoff: 763, which was 0 XIDs old when operation ended
index scan not needed: 0 pages from table (0.00% of total) had 0 dead item identifiers removed
avg read rate: 0.000 MB/s, avg write rate: 0.000 MB/s
buffer usage: 15 hits, 0 misses, 0 dirtied
WAL usage: 0 records, 0 full page images, 0 bytes
system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
INFO:  vacuuming "testdb.pg_toast.pg_toast_24597"
INFO:  finished vacuuming "testdb.pg_toast.pg_toast_24597": index scans: 0
pages: 0 removed, 0 remain, 0 scanned (100.00% of total)
tuples: 0 removed, 0 remain, 0 are dead but not yet removable
removable cutoff: 763, which was 0 XIDs old when operation ended
new relfrozenxid: 763, which is 6 XIDs ahead of previous value
index scan not needed: 0 pages from table (100.00% of total) had 0 dead item identifiers removed
avg read rate: 18.825 MB/s, avg write rate: 18.825 MB/s
buffer usage: 19 hits, 1 misses, 1 dirtied
WAL usage: 1 records, 0 full page images, 188 bytes
system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
INFO:  analyzing "public.test"
INFO:  "test": scanned 8334 of 8334 pages, containing 666667 live rows and 0 dead rows; 30000 rows in sample, 666667 estimated total rows
VACUUM
testdb=# SELECT pg_size_pretty(pg_relation_size('test')) as table_size,
pg_size_pretty(pg_relation_size('test_x_idx')) as index_size,
(pgstattuple('test')).dead_tuple_percent;
 table_size | index_size | dead_tuple_percent
------------+------------+--------------------
 65 MB      | 21 MB      |                  0
(1 row)

testdb=#
```
In our case the autovacuum was configured for the table hence the dead tuple was marked as free space. However we did run manually too also note that vacuum cannot be done for indexes hence we might need to go for second option.

2. Physical Reordering

If the vacuum could not keep up and help avoiding the bloat’s then you will have to do physical reordering of the table. The physical reordering involves rewriting the whole table. There are couple of ways in which this can be achieved.

* VACUUM FULL
VACUUM FULL will remove all bloat’s in a table and its associated indexes completely and will reclaim the disk space to OS. This will reduce the on disk table size. It does all that with rewriting the whole table,  however this is an expansive operation and it will lock the table or index for the duration of this operation, this is not desirable in most situations.

* CLUSTER
The other option is to use the CLUSTER command. This will also rewrite the table but it does that according to the specified index. Other then that it also requires to lock the table and prevents read and write operation on the table until CLUSTER operation is completed.

* pg_repack
This is an extension that is helpful in situations where VACUUM FULL or CLUSTER might not work. This extension restructures the table by creating a completely new table based on the data of the bloated table. While tracking the changes being made to the original table and will swap the new table with the original in the end.
This method does not lock the table for any read or write operations and considerably faster than VACUUM FULL or CLUSTER commands.
* REINDEX
This option can be used to remove the bloat from indexes. This command rebuilds the index specified or all indexes on the table. This option does not blocks the reads but will block the writes. However one can use the CONCURRENTLY option to avoid that but it may longer to complete than standard index creation.
```
REINDEX [ ( option [, ...] ) ] { INDEX | TABLE | SCHEMA } [ CONCURRENTLY ] name
REINDEX [ ( option [, ...] ) ] { DATABASE | SYSTEM } [ CONCURRENTLY ] [ name ]

where option can be one of:

    CONCURRENTLY [ boolean ]
    TABLESPACE new_tablespace
    VERBOSE [ boolean ]
```

Lets do the re-indexing:
```
testdb=# reindex index test_x_idx;
REINDEX
testdb=# SELECT pg_size_pretty(pg_relation_size('test')) as table_size,
pg_size_pretty(pg_relation_size('test_x_idx')) as index_size,
(pgstattuple('test')).dead_tuple_percent;
 table_size | index_size | dead_tuple_percent
------------+------------+--------------------
 65 MB      | 14 MB      |                  0
(1 row)

testdb=#
```

#### REINDEX vs. DROP INDEX & CREATE INDEX
The REINDEX statement rebuilds the index contents from the scratch, which has a similar effect as dropping and recreate of the index. However, the locking mechanisms between them are different.

##### The REINDEX statement:

* Locks writes but not reads from the table to which the index belongs.
* Takes an exclusive lock on the index that is being processed, which blocks read that attempts to use the index.

##### The DROP INDEX & CREATE INDEX statements:

* First, the DROP INDEX locks both writes and reads of the table to which the index belongs by acquiring an exclusive lock on the table.
* Then, the subsequent CREATE INDEX statement locks out writes but not reads from the index’s parent table. However, reads might be expensive during the creation of the index.
