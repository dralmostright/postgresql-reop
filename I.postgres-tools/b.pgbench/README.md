### pgbench

Pgbench is a postgresql tool that allows us to perform benchmark tests on PostgreSQL database. It allows us to perform stress tests by simulating database traffic. With these benchmark tests, it becomes possible to test different scenarios in advance and make appropriate configuration changes or resource increases. For this reason, pgbench is an important and useful tool. With pgbench, tests can be performed on the database on the current server.

pgbench is a simple program for running benchmark tests on PostgreSQL. It runs the same sequence of SQL commands over and over, possibly in multiple concurrent database sessions, and then calculates the average transaction rate (transactions per second(tps)). By default, pgbench tests a scenario that is loosely based on TPC-B, involving five SELECT, UPDATE, and INSERT commands per transaction.

You define some parameters for the workload (read-only, volume of data, number of threads, cursor sharing, …) and measure the number of transactions per second. Pgbench is used a lot when one wants to compare two alternative environments, like different postgres version, different platform, different table design,…

#### PGBENCH INSTALLATION

pgbench executable comes with ```postgresql-contrib```. Hence to install it we need to install ```postgresql-contrib```. I have already install ```postgresql-contrib``` during postgresql installation, hence we should be ok to proceed further.
```
[postgres@pgvm1 ~]$ rpm -qa | grep contrib
postgresql14-contrib-14.8-2PGDG.rhel8.x86_64
[postgres@pgvm1 ~]$
[postgres@pgvm1 ~]$ which pgbench
/usr/pgsql-14/bin/pgbench
[postgres@pgvm1 ~]$
```

Now lets create a new database:
```
postgres=# create database demodb;
CREATE DATABASE
postgres=#
```

Now lets initialize/populate the database with tables and data. And pgbench already comes up with that option:

Syntax:
```
pgbench -i [ other-options ] dbname
```

pgbench -i creates four tables pgbench_accounts, pgbench_branches, pgbench_history, and pgbench_tellers, destroying any existing tables of these names. Be very careful to use another database if you have tables having these names!

At the default scale factor of 1, the tables initially contain this many rows:
```
table                   # of rows
---------------------------------
pgbench_branches        1
pgbench_tellers         10
pgbench_accounts        100000
pgbench_history         0
```

Lets bootstrap
```
[postgres@pgvm1 ~]$ pgbench -i -s 50 demodb
dropping old tables...
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
creating tables...
generating data (client-side)...
5000000 of 5000000 tuples (100%) done (elapsed 7.92 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 11.91 s (drop tables 0.00 s, create tables 0.01 s, client-side generate 7.95 s, vacuum 1.85 s, primary keys 2.10 s).
[postgres@pgvm1 ~]$
```

The -s option is used to multiply the number of rows entered in each table. In the command above, we entered a “scaling” option of 50. This tells pgbench to create a database 50 times the default size.

This means that our pgbench_accounts table now has 5,000,000 records. It also means that our database size is approximately 800MB (50 x 16MB). Let’s check.

```
postgres=# \l+ demodb
                                                List of databases
  Name  |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges |  Size  | Tablespace | Description
--------+----------+----------+-------------+-------------+-------------------+--------+------------+-------------
 demodb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |                   | 756 MB | pg_default |
(1 row)

postgres=#
demodb=# \d+
                                         List of relations
 Schema |       Name       | Type  |  Owner   | Persistence | Access method |  Size   | Description
--------+------------------+-------+----------+-------------+---------------+---------+-------------
 public | pgbench_accounts | table | postgres | permanent   | heap          | 641 MB  |
 public | pgbench_branches | table | postgres | permanent   | heap          | 40 kB   |
 public | pgbench_history  | table | postgres | permanent   | heap          | 0 bytes |
 public | pgbench_tellers  | table | postgres | permanent   | heap          | 56 kB   |
(4 rows)

demodb=# 
```

Now lets perform the test. pgbench comes with lots of customizable options and for this test we will be using ```pgbench -c 10 -j 2 -t 10000 demodb``` where: 

The first one is -c (client), which is used to define the number of clients to connect to. For this test, we wrote 10 to tell pgbench to test with 10 clients.

Thus, pgbench opens 10 different sessions in the database while running the tests.

the next option is the number of -j (threads). This is used to define the number of running threads for pgbench. In the command below we have specified a value of 2. This will tell pgbench to start two threads during the test.

The third option is -t (transactions), the number of transactions to run. In the command below we have given the value 10,000. This means that each client session will execute 10,000 transactions.

To summarize, with this command we will simulate a total of 100,000 transactions from 10 clients.

```
[postgres@pgvm1 ~]$ pgbench -c 10 -j 2 -t 10000 demodb
pgbench (14.8)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 2
number of transactions per client: 10000
number of transactions actually processed: 100000/100000
latency average = 4.331 ms
initial connection time = 12.585 ms
tps = 2308.807106 (without initial connection time)
[postgres@pgvm1 ~]$
```

We re-iterate the test again and again by changing the parameters until we get the optimal tps. We can monitor the sessions, other performance metrics so that we can see where the improvement areas are.
