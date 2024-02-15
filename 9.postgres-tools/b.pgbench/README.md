### pgbench

Pgbench is a postgresql tool that allows us to perform benchmark tests on PostgreSQL database. It allows us to perform stress tests by simulating database traffic. With these benchmark tests, it becomes possible to test different scenarios in advance and make appropriate configuration changes or resource increases. For this reason, pgbench is an important and useful tool. With pgbench, tests can be performed on the database on the current server.

pgbench is a simple program for running benchmark tests on PostgreSQL. It runs the same sequence of SQL commands over and over, possibly in multiple concurrent database sessions, and then calculates the average transaction rate (transactions per second(tps)). By default, pgbench tests a scenario that is loosely based on TPC-B, involving five SELECT, UPDATE, and INSERT commands per transaction.

#### PGBENCH INSTALLATION

pgbench executable comes with ```postgresql-contrib```. Hence to install it we need to install ```postgresql-contrib```. I have already install ```postgresql-contrib``` during postgresql installation, hence we should be ok to proceed further.
```
[postgres@testdb ~]$ rpm -qa | grep contrib
postgresql15-contrib-15.5-2PGDG.rhel8.x86_64
[postgres@testdb ~]$
[postgres@testdb ~]$ which pgbench
/usr/pgsql-15/bin/pgbench
[postgres@testdb ~]$
```


