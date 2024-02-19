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