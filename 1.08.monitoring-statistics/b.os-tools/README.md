### Standard Unix Tools 
On most Unix platforms, PostgreSQL modifies its command title as reported by ps, so that individual server processes can readily be identified. A sample display is:
```
[postgres@pgvm1 ~]$ ps auxww | grep ^postgres | grep -v 'pts\|libex\|usr\/bin'
postgres    1379  0.0  0.6 287408 25180 ?        Ss   10:18   0:00 /usr/pgsql-14/bin/postmaster -D /var/lib/pgsql/14/data/
postgres    1457  0.0  0.1 139876  5136 ?        Ss   10:18   0:00 postgres: logger
postgres    1525  0.0  0.1 287408  6168 ?        Ss   10:18   0:00 postgres: checkpointer
postgres    1526  0.0  0.1 287408  5996 ?        Ss   10:18   0:00 postgres: background writer
postgres    1527  0.0  0.2 287408 10332 ?        Ss   10:18   0:00 postgres: walwriter
postgres    1528  0.0  0.2 288080  8956 ?        Ss   10:18   0:00 postgres: autovacuum launcher
postgres    1529  0.0  0.1 142144  5168 ?        Ss   10:18   0:00 postgres: stats collector
postgres    1530  0.0  0.1 287844  7000 ?        Ss   10:18   0:00 postgres: logical replication launcher
postgres    5138  0.0  0.2 100808 10160 ?        Ss   10:55   0:00 /usr/lib/systemd/systemd --user
postgres    5140  0.0  0.0 327740  3424 ?        S    10:55   0:00 (sd-pam)
postgres    5268  0.0  0.3 300084 14848 ?        Ss   10:55   0:00 postgres: postgres orapg [local] idle
[postgres@pgvm1 ~]$
```

The first process listed here is the primary server process. The command arguments shown for it are the same ones used when it was launched. The next 7 processes are background worker processes automatically launched by the primary process. The "autovacuum launcher" process will not be present if you have set the system not to run autovacuum. And the last remaining processes is a server process handling one client connection and there could be many in production database. Each such process sets its command line display in the form:
```
postgres: user database host activity
```
The user, database, and (client) host items remain the same for the life of the client connection, but the activity indicator changes. The activity can be idle (i.e., waiting for a client command), idle in transaction (waiting for client inside a BEGIN block), or a command type name such as SELECT. Also, waiting is appended if the server process is presently waiting on a lock held by another session.

Additionally we can use other tools like top, vmstat, iostat and there are other many more which we cna rely to monitor the performance of the database.