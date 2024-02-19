### Autovacuum/Vacuum
Autovacuum is a daemon or background utility process offered by PostgreSQL to users to issue a regular clean-up of redundant data in the database and server. It does not require the user to manually issue the vacuuming and instead, is defined in the postgresql.conf file. 

In a large-scale datacenter, the tables in the database can grow quite large. Performance can degrade significantly if stale and temporary data are not systematically removed. Vacuuming cleans up stale or temporary data in a table, and analyzing refreshes its knowledge of all the tables for the query planner.

PostgreSQL database tables are auto-vacuumed by default when 20% of the rows plus 50 rows are inserted, updated, or deleted. Tables are auto-analyzed when a threshold is met for 10% of the rows plus 50 rows. For example, a table with 10000 rows is not auto-vacuumed until 2050 rows are inserted, updated, or deleted. That same table is auto-analyzed when 1050 rows are inserted, updated, or deleted.

The default auto-vacuum analyze and vacuum settings are sufficient for a small deployment, but the percentage thresholds take longer to trigger as the tables grow larger. Performance degrades significantly before the auto-vacuum vacuuming and analyzing occurs.

It’s important to remember that VACUUM does not rewrite the block and does not "compress" the data. It simply marks the space occupied by dead tuples as "reusable." You can compress data and return the unused space back to the operating system by running VACUUM FULL.

As PostgreSQL is an MVCC architecture, because of that delete doesn't delete the rows in physical files, instead it marks them internally and this happens for both table and indexes. Hence to remove the dead tuples we will need to run vacuum or vacuum full.

There are two variants of VACUUM: standard VACUUM and VACUUM FULL. VACUUM FULL can reclaim more disk space but runs much more slowly. Also, the standard form of VACUUM can run in parallel with production database operations. (Commands such as SELECT, INSERT, UPDATE, and DELETE will continue to function normally, though you will not be able to modify the definition of a table with commands such as ALTER TABLE while it is being vacuumed.) VACUUM FULL requires an ACCESS EXCLUSIVE lock on the table it is working on, and therefore cannot be done in parallel with other use of the table. Generally, therefore, administrators should strive to use standard VACUUM and avoid VACUUM FULL.

Now we will have a demo:

First we will create a table:
```
testdb=# create table sales (sal_id integer);
CREATE TABLE
testdb=#
```

Next we will insert some data:
```
testdb=# insert into sales select g.sal_id
testdb-# from generate_series(1,10000) as g (sal_id);
INSERT 0 10000
testdb=# select count(*) from sales;
 count
-------
 10000
(1 row)

testdb=#
```

Verify Dead typles/rows:
```
testdb=# select relname, n_dead_tup from pg_stat_user_tables;
 relname | n_dead_tup
---------+------------
 sales   |          0
(1 row)

testdb=#
```
At this point we don't have dead tuples as we don't have any DML performed on it. Now lets find size taken by the table.
```
testdb=# select relname as "table_name",
testdb-# pg_size_pretty(pg_table_size(c.oid)) as "table_size"
testdb-# from
testdb-# pg_class c
testdb-# left join pg_namespace n on (n.oid=c.relnamespace)
testdb-# where nspname not in ('pg_catalog','information_schema')
testdb-# and nspname !~ '^pg_toast' and relkind in ('r')
testdb-# order by pg_table_size(c.oid)
testdb-# desc limit 1;
 table_name | table_size
------------+------------
 sales      | 392 kB
(1 row)

testdb=#
```

Now lets see the vacumm and autovacuum on that table:
```
testdb=# select relname, last_vacuum,last_autovacuum from pg_stat_user_tables;
 relname | last_vacuum |        last_autovacuum
---------+-------------+-------------------------------
 sales   |             | 2024-02-05 16:33:48.231139+04
(1 row)

testdb=#
```
The autovacuum has already run as we have enabled autovacuum.

Now lets update the table and view again the status:
```
testdb=# update sales set sal_id=sal_id+1;
UPDATE 10000
testdb=# select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
 table_name | table_size
------------+------------
 sales      | 744 kB
(1 row)

testdb=#
```
We can see the size of the table has increased by 2x.

```
testdb=# select relname, n_dead_tup from pg_stat_user_tables;
 relname | n_dead_tup
---------+------------
 sales   |          0
(1 row)

testdb=# select relname, last_vacuum,last_autovacuum from pg_stat_user_tables;
 relname | last_vacuum |        last_autovacuum
---------+-------------+-------------------------------
 sales   |             | 2024-02-05 16:44:48.485108+04
(1 row)

testdb=#
```
When querying for dead tuples, we see 0 and that is due to autovacuum. Now we will disable the auto vacuum for that table and test again.
```
testdb=# alter table sales set (autovacuum_enabled=false);
ALTER TABLE
testdb=# update sales set sal_id=sal_id+1;
UPDATE 10000
testdb=# select relname, last_vacuum,last_autovacuum from pg_stat_user_tables;
 relname | last_vacuum |        last_autovacuum
---------+-------------+-------------------------------
 sales   |             | 2024-02-05 16:44:48.485108+04
(1 row)

testdb=# select relname, n_dead_tup from pg_stat_user_tables;
 relname | n_dead_tup
---------+------------
 sales   |      10000
(1 row)

testdb=#
testdb=# select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
 table_name | table_size
------------+------------
 sales      | 744 kB
(1 row)

testdb=#
```
Now we can see dead tuples.

Here now we will run the vaccum manually:
```
testdb=# vacuum verbose analyze sales;
INFO:  vacuuming "testdb.public.sales"
INFO:  finished vacuuming "testdb.public.sales": index scans: 0
pages: 0 removed, 89 remain, 89 scanned (100.00% of total)
tuples: 10000 removed, 10000 remain, 0 are dead but not yet removable
removable cutoff: 753, which was 0 XIDs old when operation ended
new relfrozenxid: 752, which is 3 XIDs ahead of previous value
index scan not needed: 46 pages from table (51.69% of total) had 9933 dead item identifiers removed
avg read rate: 0.000 MB/s, avg write rate: 452.613 MB/s
buffer usage: 184 hits, 0 misses, 92 dirtied
WAL usage: 182 records, 48 full page images, 93457 bytes
system usage: CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s
INFO:  analyzing "public.sales"
INFO:  "sales": scanned 89 of 89 pages, containing 10000 live rows and 0 dead rows; 10000 rows in sample, 10000 estimated total rows
VACUUM
testdb=#
```

Now lets verify the dead tuples and size:
```
testdb=# select relname, n_dead_tup from pg_stat_user_tables;
 relname | n_dead_tup
---------+------------
 sales   |          0
(1 row)

testdb=# select relname, last_vacuum,last_autovacuum from pg_stat_user_tables;
 relname |          last_vacuum          |        last_autovacuum
---------+-------------------------------+-------------------------------
 sales   | 2024-02-05 16:48:52.776638+04 | 2024-02-05 16:44:48.485108+04
(1 row)

testdb=# select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
 table_name | table_size
------------+------------
 sales      | 744 kB
(1 row)

testdb=#
```

As we can see the vaccum marked dead tuples as free space which database can put next coming tuples. However if we want to compact the table and release the space to O/S we need to do full vacuum and this locks the table in exclusive mode:
```
testdb=# select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
 table_name | table_size
------------+------------
 sales      | 744 kB
(1 row)

testdb=# vacuum full verbose analyze sales;
INFO:  vacuuming "public.sales"
INFO:  "public.sales": found 0 removable, 10000 nonremovable row versions in 89 pages
DETAIL:  0 dead row versions cannot be removed yet.
CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.01 s.
INFO:  analyzing "public.sales"
INFO:  "sales": scanned 45 of 45 pages, containing 10000 live rows and 0 dead rows; 10000 rows in sample, 10000 estimated total rows
VACUUM
testdb=#
testdb=# select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
 table_name | table_size
------------+------------
 sales      | 360 kB
(1 row)

testdb=#
```
As we can see the size of the table has been reduced. The full vacuum re-rwites the pages and organize them.

Some useful queries:
```
select relname as "table_name",
pg_size_pretty(pg_table_size(c.oid)) as "table_size"
from 
pg_class c
left join pg_namespace n on (n.oid=c.relnamespace)
where nspname not in ('pg_catalog','information_schema')
and nspname !~ '^pg_toast' and relkind in ('r')
order by pg_table_size(c.oid)
desc limit 1;
```
