### PostgreSQL logfile
Whenever troubleshooting the main log file where postgresql throws info, warning and errors will be highly beneficial. And we can find the PostgreSQL log file using below:
```
[postgres@pgvm1 ~]$ psql -c 'SELECT  pg_current_logfile();'
   pg_current_logfile
------------------------
 log/postgresql-Sun.log
(1 row)

[postgres@pgvm1 ~]$
```
This will work for most of the higher versions and the directory is relative to the data directory which in our case is :
```
[postgres@pgvm1 ~]$ env | grep PG
PGDATA=/var/lib/pgsql/14/data
[postgres@pgvm1 ~]$
```
But if you are in lower version we will need to use the below apporach:
```
[postgres@pgvm1 ~]$ psql -c 'SHOW logging_collector;'
 logging_collector
-------------------
 on
(1 row)

[postgres@pgvm1 ~]$
```
We need to verify that the logging collector is started as above, then if its on we can fire below command:
```
[postgres@pgvm1 ~]$ psql -c 'show log_directory'
 log_directory
---------------
 log
(1 row)

[postgres@pgvm1 ~]$
```
Again its the relative directory from data directory:

And in case the databaase is not up and running we need to rely on OS commands like find, locate to find the log files.
```
[postgres@pgvm1 ~]$ find / -type f -name "postgresql*.log"  2>/dev/null
/var/lib/pgsql/14/data/log/postgresql-Sun.log
/var/lib/pgsql/14/data/log/postgresql-Sat.log
/var/lib/pgsql/14/data/log/postgresql-Mon.log
[postgres@pgvm1 ~]$
```