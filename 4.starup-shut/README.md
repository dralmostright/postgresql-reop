### Starting the Database Server
Before anyone can access the database, you must start the database server. The database server program is called postgres. If you are using a pre-packaged version of PostgreSQL, it almost certainly includes provisions for running the server as a background task according to the conventions of your operating system. Even though we have used pre-packaged version we will try to explore the options:

The bare-bones way to start the server manually is just to invoke postgres directly, specifying the location of the data directory with the -D option, for example:
```
[postgres@pgvm1 ~]$ postgres -D /var/lib/pgsql/14/data
2024-02-04 08:54:17.903 EST [8506] LOG:  redirecting log output to logging collector process
2024-02-04 08:54:17.903 EST [8506] HINT:  Future log output will appear in directory "log".
```
Which will leave the server running in the foreground and whenever the sessions is gone, database will be shutdown. This must be done while logged into the PostgreSQL user account.

And the log directory will be ```log``` relative to datadir ```/var/lib/pgsql/14/data``` and we can see below logs:

```
[postgres@pgvm1 log]$ tail -f postgresql-Sun.log
2024-02-04 08:51:23.749 EST [1393] LOG:  database system is shut down
2024-02-04 08:54:17.903 EST [8506] LOG:  starting PostgreSQL 14.8 on x86_64-pc-linux-gnu, compil ed by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18), 64-bit
2024-02-04 08:54:17.904 EST [8506] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2024-02-04 08:54:17.904 EST [8506] LOG:  listening on IPv6 address "::", port 5432
2024-02-04 08:54:17.912 EST [8506] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL. 5432"
2024-02-04 08:54:17.913 EST [8506] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2024-02-04 08:54:17.915 EST [8508] LOG:  database system was shut down at 2024-02-04 08:51:23 ES T
2024-02-04 08:54:17.918 EST [8506] LOG:  database system is ready to accept connections
```

Addtionally if you want to see where the log file has been located, we can use following:
```
postgres=# SELECT pg_current_logfile();
   pg_current_logfile
------------------------
 log/postgresql-Sun.log
(1 row)

postgres=#
```

Normally it is better to start postgres in the background. For this, use the usual Unix shell syntax:
```
[postgres@pgvm1 ~]$ postgres -D /var/lib/pgsql/14/data  >> /var/lib/pgsql/14/data/log/postgresql-Sun.log 2>&1 &
[1] 8671
[postgres@pgvm1 ~]$
```

This shell syntax can get tedious quickly. Therefore the wrapper program pg_ctl is provided to simplify some tasks. For example:
```
[postgres@pgvm1 ~]$ pg_ctl start -l /var/lib/pgsql/14/data/log/postgresql-Sun.log
pg_ctl: another server might be running; trying to start server anyway
waiting for server to start.... done
server started
[postgres@pgvm1 ~]$ 
```
will start the server in the background and put the output into the named log file. The -D option has the same meaning here as for postgres. pg_ctl is also capable of stopping the server, which we will explore in next sections:

Different systems have different conventions for starting up daemons at boot time. Many systems have a file /etc/rc.local or /etc/rc.d/rc.local. Others use init.d or rc.d directories. Whatever you do, the server must be run by the PostgreSQL user account and not by root or any other user. 

When using systemd, you can use the following service unit file (e.g., postgresql.service):

```
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
User=postgres
ExecStart=/usr/local/pgsql/bin/postgres -D /usr/local/pgsql/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=infinity

[Install]
WantedBy=multi-user.target
```

### Shutting Down Database Server
We can choose to shutdown PostgreSQL in 3 modes:

* Smart mode
* Fast mode
* Immediate mode

We can use the flag -m to invoke PostgreSQL shutdown using a specific mode.

#### Smart mode(-ms)(SIGTERM): 
This is the default mode; hence just a stop should work. Explicitly invoking shutdown using smart mode is actually not needed. After receiving SIGTERM, the server disallows new connections, but lets existing sessions end their work normally. It shuts down only after all of the sessions terminate. If the server is in recovery when a smart shutdown is requested, recovery and streaming replication will be stopped only after all regular sessions have terminated.
```
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$

[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop -ms
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$
```

#### Fast mode(-mf) (SIGINT): 
To stop PostgreSQL in fast mode, we should use -mf as shown below. The server disallows new connections and sends all existing server processes SIGTERM, which will cause them to abort their current transactions and exit promptly. It then waits for all server processes to exit and finally shuts down.
```
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop -mf
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$
```

#### Immediate mode(-mi)(SIGQUIT): 
We must use -mi to stop the PostgreSQL server using immediate mode. The server will send SIGQUIT to all child processes and wait for them to terminate. If any do not terminate within 5 seconds, they will be sent SIGKILL. The supervisor server process exits as soon as all child processes have exited, without doing normal database shutdown processing. This will lead to recovery (by replaying the WAL log) upon next start-up. This is recommended only in emergencies.
```
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop -mi
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$
```

It is best not to use SIGKILL to shut down the server. Doing so will prevent the server from releasing shared memory and semaphores. Furthermore, SIGKILL kills the postgres process without letting it relay the signal to its subprocesses, so it might be necessary to kill the individual subprocesses by hand as well.

If you have been shutting down the postgres with systemd then the parameters ```KillSignal``` defined in postgresql.service will determine which mode to shutdown the database.