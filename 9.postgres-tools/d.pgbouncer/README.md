### pgBouncer

PgBouncer is an open-source, single-binary, lightweight Postgres Pro connection pooler for PostgreSQL. PgBouncer can manage connection pools for PostgreSQL clusters. In the world of modern database management, the efficient handling of client connections is crucial for ensuring optimal performance and scalability and this is where PgBouncer comes into play:

PgBouncer acts as a middleman between the application and the PostgreSQL server, providing a connection pooling mechanism. By managing a pool of pre-established connections, PgBouncer reduces the overhead of repeatedly creating and tearing down connections for each client, resulting in improved response times and resource utilization.

#### Installation of PgBouncer
The installation process may vary based on your operating system, but package managers like `apt` or `yum` can streamline the process.
```
[root@pgvm1 ~]# yum install pgbouncer
Last metadata expiration check: 18:16:28 ago on Sun 18 Feb 2024 05:31:00 PM +0545.
Dependencies resolved.
================================================================================
 Package              Arch       Version                  Repository       Size
================================================================================
Installing:
 pgbouncer            x86_64     1.22.0-42PGDG.rhel8      pgdg-common     262 k
Installing dependencies:
 python3-psycopg2     x86_64     2.9.5-3.rhel8            pgdg-common     188 k

Transaction Summary
================================================================================
Install  2 Packages

Total download size: 449 k
Installed size: 1.2 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): python3-psycopg2-2.9.5-3.rhel8.x86_64.rp 122 kB/s | 188 kB     00:01
(2/2): pgbouncer-1.22.0-42PGDG.rhel8.x86_64.rpm 170 kB/s | 262 kB     00:01
--------------------------------------------------------------------------------
Total                                           291 kB/s | 449 kB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : python3-psycopg2-2.9.5-3.rhel8.x86_64                  1/2
  Running scriptlet: pgbouncer-1.22.0-42PGDG.rhel8.x86_64                   2/2
  Installing       : pgbouncer-1.22.0-42PGDG.rhel8.x86_64                   2/2
  Running scriptlet: pgbouncer-1.22.0-42PGDG.rhel8.x86_64                   2/2
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

  Verifying        : pgbouncer-1.22.0-42PGDG.rhel8.x86_64                   1/2
  Verifying        : python3-psycopg2-2.9.5-3.rhel8.x86_64                  2/2

Installed:
  pgbouncer-1.22.0-42PGDG.rhel8.x86_64   python3-psycopg2-2.9.5-3.rhel8.x86_64

Complete!
[root@pgvm1 ~]#
```

#### Configure PgBouncer
PgBouncer's main configuration file is usually named pgbouncer.ini, which is typically found in ```/etc/pgbouncer/pgbouncer.ini```. You should edit this file to set up the connection parameters for your PostgreSQL database and define user authentication details. Additionally, you can configure connection pooling settings, timeouts, and other options. Pay close attention to the following parameters:

* ```listen_addr```: The IP address or hostname on which PgBouncer listens for incoming connections.
* ```listen_port```: The port number on which PgBouncer listens for incoming connections.
* ```auth_type```: Set it to md5 or trust (for testing purposes only). You can also use a configuration file named userlist.txt to define usernames and passwords.
* ```pool_mode```: Set it to transaction or session mode, depending on your application's requirements.
* ```default_pool_size```: The default number of connections PgBouncer will pool.
* ```max_client_conn```: The maximum number of client connections allowed.
* ```min_pool_size```: The minimum number of connections to keep in the pool, even when idle.
* ```reserve_pool_size```: The number of additional connections to allow when the pool is under heavy load.
* ```server_reset_query```: If you use a connection-level setting, specify a reset query to ensure the connection is clean when returned to the pool.

Now lets make the changes in the configuration files as below:
```
[root@pgvm1 ~]# vi /etc/pgbouncer/pgbouncer.ini 
[root@pgvm1 ~]# cat /etc/pgbouncer/pgbouncer.ini | grep 'port\|pool_mode\|max_client_conn\|auth_type\|auth_file\|admin_users' | grep -v ';'
* = host=localhost port=5432
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
admin_users = postgres
pool_mode = transaction
max_client_conn = 5000
[root@pgvm1 ~]#
```
Now let’s open the file with users at /etc/pgbouncer/userlist.txt and to determine which hashing algoright is used we can use below query:
```
[postgres@pgvm1 ~]$ psql -U postgres -d postgres -c "SELECT concat('\"', usename, '\" \"', passwd, '\"') FROM pg_shadow"
                                                                       concat
----------------------------------------------------------------------------------------------------------------------------------------------------
 "testusr1" "SCRAM-SHA-256$4096:YGL3R4s38tjHwPQZhLXkfw==$7wkXI1esMP/CRhcQmLNm7Ex+GIMJC4b0TrTmi8sngP4=:sqvPOlnSL7zU3mxSlEdpqo/kq3j7zdN3tspyWipsV5U="
 "postgres" "SCRAM-SHA-256$4096:fOF4mQWQhMp3f2HgUCvBEg==$m5xCBtbM89nCacWgNgcFTNK58VJQ+MtbXyhH8PdgeRI=:bZgLj+Lc9A+ENJEyFJU2KRzJVkp5CtGilYdWJgcusjg="
(2 rows)

[postgres@pgvm1 ~]$
```

Place the username in double quotes and the scram-sha-256 password hash (in one line):
```
[root@pgvm1 ~]# vi /etc/pgbouncer/userlist.txt
[root@pgvm1 ~]# cat /etc/pgbouncer/userlist.txt
"postgres" "SCRAM-SHA-256$4096:fOF4mQWQhMp3f2HgUCvBEg==$m5xCBtbM89nCacWgNgcFTNK58VJQ+MtbXyhH8PdgeRI=:bZgLj+Lc9A+ENJEyFJU2KRzJVkp5CtGilYdWJgcusjg="
[root@pgvm1 ~]#
```

To make our application use PgBouncer when connecting to the database, the only change that is needed is replacing the port number: use 6432 instead of 5432.

With all this changes lets start the pgBouncer:
```
[root@pgvm1 ~]# systemctl status pgbouncer
● pgbouncer.service - A lightweight connection pooler for PostgreSQL
   Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@pgvm1 ~]# systemctl start pgbouncer
[root@pgvm1 ~]# systemctl status pgbouncer
● pgbouncer.service - A lightweight connection pooler for PostgreSQL
   Loaded: loaded (/usr/lib/systemd/system/pgbouncer.service; disabled; vendor preset: disabled)
   Active: active (running) since Mon 2024-02-19 12:12:35 +0545; 1s ago
 Main PID: 3941 (pgbouncer)
    Tasks: 2 (limit: 22960)
   Memory: 1.1M
   CGroup: /system.slice/pgbouncer.service
           └─3941 /usr/bin/pgbouncer /etc/pgbouncer/pgbouncer.ini

Feb 19 12:12:35 pgvm1.localdomain systemd[1]: Starting A lightweight connection pooler for PostgreSQL...
Feb 19 12:12:35 pgvm1.localdomain pgbouncer[3941]: kernel file descriptor limit: 1024 (hard: 262144); max_client_conn: 5000, max expected fd use: 5012
Feb 19 12:12:35 pgvm1.localdomain pgbouncer[3941]: listening on [::1]:6432
Feb 19 12:12:35 pgvm1.localdomain pgbouncer[3941]: listening on 127.0.0.1:6432
Feb 19 12:12:35 pgvm1.localdomain pgbouncer[3941]: listening on unix:/tmp/.s.PGSQL.6432
Feb 19 12:12:35 pgvm1.localdomain pgbouncer[3941]: process up: PgBouncer 1.22.0, libevent 2.1.8-stable (epoll), adns: evdns2, tls: OpenSSL 1.1.1g FIPS  21 Apr 2020
Feb 19 12:12:35 pgvm1.localdomain systemd[1]: Started A lightweight connection pooler for PostgreSQL.
[root@pgvm1 ~]# netstat -tupln | grep 6432
tcp        0      0 127.0.0.1:6432          0.0.0.0:*               LISTEN      3941/pgbouncer
tcp6       0      0 ::1:6432                :::*                    LISTEN      3941/pgbouncer
[root@pgvm1 ~]#
```

Let’s run a performance test to compare the performance of connecting to PostgreSQL with and without PgBouncer, using the pgbench utility, before doing that lets verify the connectivity but connecting on different port 6432.
```
[postgres@pgvm1 ~]$ psql -U postgres -h localhost -p 6432 -d postgres -c "SELECT concat('\"', usename, '\" \"', passwd, '\"') FROM pg_shadow"
Password for user postgres:
                                                                       concat
----------------------------------------------------------------------------------------------------------------------------------------------------
 "testusr1" "SCRAM-SHA-256$4096:YGL3R4s38tjHwPQZhLXkfw==$7wkXI1esMP/CRhcQmLNm7Ex+GIMJC4b0TrTmi8sngP4=:sqvPOlnSL7zU3mxSlEdpqo/kq3j7zdN3tspyWipsV5U="
 "postgres" "SCRAM-SHA-256$4096:fOF4mQWQhMp3f2HgUCvBEg==$m5xCBtbM89nCacWgNgcFTNK58VJQ+MtbXyhH8PdgeRI=:bZgLj+Lc9A+ENJEyFJU2KRzJVkp5CtGilYdWJgcusjg="
(2 rows)

[postgres@pgvm1 ~]$
```

Now we will try to run the test with 1000 connections but the database has around 100:
```
[postgres@pgvm1 ~]$ psql -U postgres -h localhost -p 6432 -d postgres -c "show max_connections;"
Password for user postgres:
 max_connections
-----------------
 100
(1 row)

[postgres@pgvm1 ~]$
```
Connecting to the Postgres database server without using PgBouncer. This command will start a test with 1000 concurrent clients for 60 seconds, connecting directly to the PostgreSQL database.
```
[postgres@pgvm1 ~]$ pgbench -c 1000 -T 60 demodb -h 127.0.0.1 -p 5432 -U postgres
Password:
pgbench (14.8)
starting vacuum...end.
pgbench: error: connection to server at "127.0.0.1", port 5432 failed: FATAL:  sorry, too many clients already
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 1000
number of threads: 1
duration: 60 s
number of transactions actually processed: 0
pgbench: fatal: Run was aborted; the above results are incomplete.
[postgres@pgvm1 ~]$
```

When connecting to the database using PgBouncer, everything works without any issues.
```
[postgres@pgvm1 ~]$ pgbench -c 1000 -T 30 demodb -h 127.0.0.1 -p 6432 -U postgres
Password:
pgbench (14.8)
starting vacuum...end.
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 1000
number of threads: 1
duration: 30 s
number of transactions actually processed: 24408
latency average = 848.964 ms
initial connection time = 10064.164 ms
tps = 1177.906550 (without initial connection time)
[postgres@pgvm1 ~]$
```

Let’s compare a latency and a number of transactions per second (tps) that the database performs when the application connects to the database without using PgBouncer and when it uses PgBouncer.

The following test performs select-only transactions.
```
[postgres@pgvm1 ~]$ vi pgsql.sql
[postgres@pgvm1 ~]$ cat pgsql.sql
select 1;
[postgres@pgvm1 ~]$
```

The -C option in the pgbench indicates that for every single transaction, pgbench will close the open connection and create a new one. This is useful for measuring the connection overhead.

Below test is without using pgBouncer:
```
[postgres@pgvm1 ~]$ pgbench -c 20 -t 100 -S demodb -h 127.0.0.1 -p 5432 -U postgres -C -f pgsql.sql
Password:
pgbench (14.8)
starting vacuum...end.
transaction type: multiple scripts
scaling factor: 50
query mode: simple
number of clients: 20
number of threads: 1
number of transactions per client: 100
number of transactions actually processed: 2000/2000
latency average = 243.656 ms
average connection time = 12.156 ms
tps = 82.082900 (including reconnection times)
SQL script 1: <builtin: select only>
 - weight: 1 (targets 50.0% of total)
 - 1016 transactions (50.8% of total, tps = 41.698113)
 - latency average = 112.556 ms
 - latency stddev = 68.505 ms
SQL script 2: pgsql.sql
 - weight: 1 (targets 50.0% of total)
 - 984 transactions (49.2% of total, tps = 40.384787)
 - latency average = 115.684 ms
 - latency stddev = 68.192 ms
[postgres@pgvm1 ~]$
```

Test using pgBouncer:
```
[postgres@pgvm1 ~]$ pgbench -c 20 -t 100 -S demodb -h 127.0.0.1 -p 6432 -U postgres -C -f pgsql.sql
Password:
pgbench (14.8)
starting vacuum...end.
transaction type: multiple scripts
scaling factor: 50
query mode: simple
number of clients: 20
number of threads: 1
number of transactions per client: 100
number of transactions actually processed: 2000/2000
latency average = 210.990 ms
average connection time = 10.530 ms
tps = 94.791380 (including reconnection times)
SQL script 1: <builtin: select only>
 - weight: 1 (targets 50.0% of total)
 - 1000 transactions (50.0% of total, tps = 47.395690)
 - latency average = 102.205 ms
 - latency stddev = 59.441 ms
SQL script 2: pgsql.sql
 - weight: 1 (targets 50.0% of total)
 - 1000 transactions (50.0% of total, tps = 47.395690)
 - latency average = 99.248 ms
 - latency stddev = 60.420 ms
[postgres@pgvm1 ~]$
```

If we compare latency we can easily see the improvement brought by pgBouncer.