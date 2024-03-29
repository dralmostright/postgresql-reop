### PG Center

pgCenter is a command-line admin tool for observing and troubleshooting PostgreSQL i.e monitor PostgreSQL cluster. Postgres provides various activity statistics about its runtime, such as connections, statements, database operations, replication, resources usage, and more. The general purpose of the statistics is to help DBAs to monitor and troubleshoot Postgres. However, these statistics are provided in textual form retrieved from SQL functions and views, and Postgres doesn’t provide native tools for working with statistics views.

pgCenter’s main goal is to help Postgres DBA work with statistics and provide a convenient way to observe Postgres in runtime. [pgcenter](https://github.com/lesovsky/pgcenter) is an admin tool for working with PostgreSQL stats, written in Golang. It provides top like viewer with a few admin functions, tool for recording stats into files and building reports.

#### Installing pgCenter package
We can install pgcenter using yum as below:
```
[root@testdb ~]# yum install https://github.com/lesovsky/pgcenter/releases/download/v0.9.2/pgcenter_0.9.2_linux_amd64.rpm
Last metadata expiration check: 0:14:06 ago on Tue 20 Feb 2024 07:59:37 PM +0545.
pgcenter_0.9.2_linux_amd64.rpm                    1.1 MB/s | 5.6 MB     00:05
Dependencies resolved.
==================================================================================
 Package           Architecture    Version            Repository             Size
==================================================================================
Installing:
 pgcenter          x86_64          0.9.2-1            @commandline          5.6 M

Transaction Summary
==================================================================================
Install  1 Package

Total size: 5.6 M
Installed size: 11 M
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                          1/1
  Installing       : pgcenter-0.9.2-1.x86_64                                  1/1
  Verifying        : pgcenter-0.9.2-1.x86_64                                  1/1

Installed:
  pgcenter-0.9.2-1.x86_64

Complete!
[root@testdb ~]#
```

Now lets verify if its accessiable or not:
```
[postgres@testdb ~]$ which pgcenter
/usr/bin/pgcenter
[postgres@testdb ~]$ pgcenter --version
pgcenter v0.9.2 7ebd54847cbbf72a98cbde39e9afc3845cba9839-release
[postgres@testdb ~]$
[postgres@testdb ~]$ pgcenter top --help
'pgcenter top' is the top-like stats viewer.

Usage:
  pgcenter top [OPTIONS]... [DBNAME [USERNAME]]

Options:
  -d, --dbname DBNAME           database name to connect to
  -h, --host HOSTNAME           database server host or socket directory
  -p, --port PORT               database server port (default 5432)
  -U, --username USERNAME       database user name

General options:
  -?, --help            show this help and exit

Report bugs to <https://github.com/lesovsky/pgcenter/issues>.
[postgres@testdb ~]$
```

Now lets view the top using below cmd:
```
[postgres@testdb ~]$ pgcenter top -d testdb -h localhost -U postgres  -p 5432
Password for user postgres:
```

Once proper credentials and info is provided below is the output the above command will generate:
```
pgcenter: 2024-02-20 20:18:46, load average: 0.00, 0.03, 0.05                                  state [ok]: localhost:5432 postgres@testdb (ver: 15.5, up 01:20:09, recovery: f)
    %cpu:  1.0 us,  2.0 sy,  0.0 ni, 95.0 id,  0.0 wa,  2.0 hi,  0.0 si,  0.0 st                 activity:  1/100 conns,  0/0 prepared,  0 idle,  0 idle_xact,  1 active,  0 waiting,  0 othe
 MiB mem:   1975 total,   1065 free,    117 used,      793 buff/cached                         autovacuum:  0/3 workers/max,  0 manual,  0 wraparound, 00:00:00 vac_maxtime
MiB swap:   2047 total,   2047 free,      0 used,      0/0 dirty/writeback                     statements:   0 stmt/s, 0.000 stmt_avgtime, 00:00:00 xact_maxtime, 00:00:00 prep_maxtime

pid       cl_addr       cl_port   datname   usename   appname   backend_type    wait_etype  wait_event  state     xact_age  query_age  change_age  query
1816      127.0.0.1/32  33516     testdb    postgres            client backend                          active    00:00:00  00:00:00   00:00:00    SELECT pid, client_addr AS cl_addr, client
92639                   -1        sample1   postgres  psql      client backend                          active    00:00:42  00:00:42   00:00:42    insert into t_test v
```

pgCenter profile command: pgcenter profile is the tool for profiling wait events occurred during queries execution. In cases of long queries, you might be interested what this query does. Using EXPLAIN utility you can observe a detailed query execution plan. But if the query spends time waiting’s EXPLAIN will not show that. Using pgcenter profile you can see what wait events occur during query execution. Please note -P 92639 is same insert PID from above top.
```
[postgres@testdb ~]$ pgcenter profile -d testdb -h localhost -U postgres -p 5432 -P 92639
Password for user postgres:

LOG: Profiling process 92639 with 100ms sampling
------ ------------ -----------------------------
% time      seconds wait_event                     query: insert into t_test values (generate_series(1,100000000),now());
------ ------------ -----------------------------
 
 23.74   100.816840 IO.WALInitWrite
 23.71   100.693757 Running
 22.86    97.062093 IO.DataFileExtend
 18.82    79.920575 LWLock.WALWrite
  5.70    24.183814 IO.WALInitSync
  4.01    17.028502 IO.WALSync
  0.70     2.956554 IO.WALWrite
  0.42     1.773102 IO.DataFileWrite
  0.05     0.193801 LWLock.CheckpointerComm
------ ------------ -----------------------------
100.00   424.629038 including workers
         424.629038

[postgres@testdb ~]$ 
```

pgCenter record command: pgcenter record collect metrics from Postgres to local files. pgcenter record connects to Postgres, reads stats, and writes this information into JSON files into a tar archive. File name contains the name of the statistics view and timestamp when stats have been recorded. Hence, it’s possible to unpack statistics using tar. Once unpacked, stats can be used in any way required.

pgCenter report command: pgcenter report reads stats files written by pgcenter record and builds reports. pgcenter report is an addition to pgcenter record. It reads the collected statistics and builds reports out of these data. pgcenter report doesn’t require connection to Postgres, all you need is to specify the file with relevant statistics and choose the type of the report.
