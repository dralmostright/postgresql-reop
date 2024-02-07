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


