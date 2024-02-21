### PostgreSQL Replication Switchover

PostgreSQL replication plays a vital role in ensuring high availability and data redundancy for your database. Today we are going to outline the step-by-step process of performing a manual switchover in PostgreSQL.

Verify Standby Synchronization:
On Primary:
```
postgres=# select case
postgres-#            when pg_is_in_recovery() = 'f' Then 'Primary'
postgres-#        else 'Standby'
postgres-#    END Database_role,
postgres-#    pg_read_file('/etc/hostname') hostname;
-[ RECORD 1 ]-+------------------
database_role | Primary
hostname      | pgvm1.localdomain+
              |

postgres=#
```

On Standby:
```
postgres=# select case
postgres-#            when pg_is_in_recovery() = 'f' Then 'Primary'
postgres-#        else 'Standby'
postgres-#    END Database_role,
postgres-#    pg_read_file('/etc/hostname') hostname;
 database_role |     hostname
---------------+-------------------
 Standby       | pgvm2.localdomain+
               |
(1 row)

postgres=#
```

On Standby:
```
postgres=# SELECT CASE
postgres-#     WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() THEN 0
postgres-#     ELSE EXTRACT(EPOCH FROM now() - pg_last_xact_replay_timestamp())
postgres-# END AS log_delay;
-[ RECORD 1 ]
log_delay | 0

postgres=# 
```
As we can see the delay is 0 hence we will proceed to next step:

Allow Current Live IP in Current Standby:

To permit the current live IP to access the current backup (standby), modify the pg_hba.conf file:
```
[postgres@pgvm2 data]$ vi pg_hba.conf
[postgres@pgvm2 data]$ cat pg_hba.conf | grep trust
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
host    replication     all             192.168.229.138/32      trust
[postgres@pgvm2 data]$
```

Shutdown the Primary:
Make sure all the application sessions are drained and shutdown applications as well.
```
[postgres@pgvm1 ~]$ pg_ctl stop -ms
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 ~]$
```

Promote the Standby to Read-Write
```
[postgres@pgvm2 ~]$ psql -c "SELECT pg_promote();"
 pg_promote
------------
 t
(1 row)

[postgres@pgvm2 ~]$
```

Create standby.signal on Old Primary

Create the standby.signal file on the old primary to ensure it starts as a standby when brought back online:
```
[postgres@pgvm1 data]$ pwd
/var/lib/pgsql/14/data
[postgres@pgvm1 data]$ touch standby.signal
[postgres@pgvm1 data]$
```

Edit postgresql.auto.conf on Old Primary
```
[postgres@pgvm1 data]$ vi postgresql.auto.conf
[postgres@pgvm1 data]$ cat postgresql.auto.conf | grep primary
primary_conninfo = 'user=postgres passfile=''/var/lib/pgsql/.pgpass'' channel_binding=prefer host=192.168.229.140 port=5432 sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
[postgres@pgvm1 data]$
```

Now lets start the database:
```
[postgres@pgvm1 ~]$ pg_ctl start
waiting for server to start....2024-02-21 06:50:29 EST [16856]: [1-1] user=,db=,app=,client= LOG:  redirecting log output to logging collector process
2024-02-21 06:50:29 EST [16856]: [2-1] user=,db=,app=,client= HINT:  Future log output will appear in directory "log".
 done
server started
[postgres@pgvm1 ~]$
```

Verify the Switchover:
On Old Primary:
```
postgres=# select case
postgres-#            when pg_is_in_recovery() = 'f' Then 'Primary'
postgres-#        else 'Standby'
postgres-#    END Database_role,
postgres-#    pg_read_file('/etc/hostname') hostname;
 database_role |     hostname
---------------+-------------------
 Standby       | pgvm1.localdomain+
               |
(1 row)

postgres=#
```

On New Primary:
```
postgres=# select case
postgres-#            when pg_is_in_recovery() = 'f' Then 'Primary'
postgres-#        else 'Standby'
postgres-#    END Database_role,
postgres-#    pg_read_file('/etc/hostname') hostname;
 database_role |     hostname
---------------+-------------------
 Primary       | pgvm2.localdomain+
               |
(1 row)

postgres=#
```

Note:
Make sure the log for both instances are clean and fix the errors if any else after switchover streaming may not occur.

On Old Standby:
```
postgres=# SELECT CASE
postgres-#     WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() THEN 0
postgres-#     ELSE EXTRACT(EPOCH FROM now() - pg_last_xact_replay_timestamp())
postgres-# END AS log_delay;
 log_delay
-----------
         0
(1 row)

postgres=#
```

Now lets create a new database on new primary:
```
postgres=# create database hellodb;
CREATE DATABASE
postgres=# \l hellodb
                               List of databases
  Name   |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges
---------+----------+----------+-------------+-------------+-------------------
 hellodb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(1 row)

postgres=#
```

And verify the same in new Standby:
```
postgres=# \l hellodb
                               List of databases
  Name   |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges
---------+----------+----------+-------------+-------------+-------------------
 hellodb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(1 row)

postgres=#
```