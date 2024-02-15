### Restore and Recovery

#### Point in time recovery in Postgresql

In this tutorail we are going to duscuss Point in time recovery options in Postgresql:

We will use backup created by pg_basebackup for PITR

Lest switch the wal in postgres:
```
orapg=# SELECT pg_switch_wal();
 pg_switch_wal
---------------
 0/7050030
(1 row)

orapg=#
```

The WAL location indicated by LSN ```0/7050030``` is the point of time that we would like to recover up to in our example.

Lets stop the database and remove the contents from data directory:
```
[postgres@pgvm1 ~]$ pg_ctl -D $PGDATA stop
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 ~]$
[postgres@pgvm1 data]$ pwd
/var/lib/pgsql/14/data
[postgres@pgvm1 data]$ rm -rf *
[postgres@pgvm1 data]$ ls
[postgres@pgvm1 data]$
```

Now lets resore the backup:

```
[postgres@pgvm1 data]$ cd /pgdata/pgbkp/15022024/
[postgres@pgvm1 15022024]$ cp -rpv  * /var/lib/pgsql/14/data
'backup_label' -> '/var/lib/pgsql/14/data/backup_label'
'backup_manifest' -> '/var/lib/pgsql/14/data/backup_manifest'
'base' -> '/var/lib/pgsql/14/data/base'
'base/1' -> '/var/lib/pgsql/14/data/base/1'
.....
.....
.....
'postgresql.auto.conf' -> '/var/lib/pgsql/14/data/postgresql.auto.conf'
'postgresql.conf' -> '/var/lib/pgsql/14/data/postgresql.conf'
[postgres@pgvm1 15022024]$
```

Lets edit postgresql.conf and set recover target. Since we are using LSN as our target, we can simply put the LSN we captured to recovery_target_lsn configuration. However there are other options where we can use exact time and other options too. Lets update the below two parameters on postgresql.conf file.
```
[postgres@pgvm1 data]$ cat postgresql.conf | grep recovery_target_lsn
recovery_target_lsn = '0/7050030'       # the WAL LSN up to which recovery will proceed
[postgres@pgvm1 data]$ cat postgresql.conf | grep restore_comm
restore_command = ' /walarc/pg14/archive/%f %p'         # command to use to restore an archived logfile segment
[postgres@pgvm1 data]$
```

Signal the database to run in recovery mode by creating a recovery.signal file under the pgtest cluster
```
[postgres@pgvm1 data]$ touch recovery.signal
[postgres@pgvm1 data]$
```
Lets start the server:
```
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA start
waiting for server to start....2024-02-15 04:39:55.642 EST [6261] LOG:  redirecting log output to logging collector process
2024-02-15 04:39:55.642 EST [6261] HINT:  Future log output will appear in directory "log".
 done
server started
[postgres@pgvm1 data]$
```
The server will now start in recovery mode and it will restore WAL files from the archive and perform the recover.

You may notice that even though the database has been recovered to a point of time in the past, you will encounter a database in recovery or read only database error if you intend to insert additional data. This is because we are still in the recovery mode but is currently paused.

This is configured by the recovery_target_action option, which defaults to pause. This is actually intended, to allow you to have a moment to check your database and confirm that it is indeed the database state that you would like to recover to. If this is wrong, you can simply shutdown the database and reconfigure the recovery_target_lsn until you reach a desired state of database.

Below is the excerpt of the log file.
```
2024-02-15 04:39:55.642 EST [6261] LOG:  starting PostgreSQL 14.8 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-18), 64-bit
2024-02-15 04:39:55.643 EST [6261] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2024-02-15 04:39:55.643 EST [6261] LOG:  listening on IPv6 address "::", port 5432
2024-02-15 04:39:55.644 EST [6261] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2024-02-15 04:39:55.645 EST [6261] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5432"
2024-02-15 04:39:55.650 EST [6263] LOG:  database system was interrupted; last known up at 2024-02-15 04:08:50 EST
cp: cannot stat '/walarc/pg14/archive/00000002.history': No such file or directory
2024-02-15 04:39:55.660 EST [6263] LOG:  starting point-in-time recovery to WAL location (LSN) "0/7050030"
2024-02-15 04:39:55.669 EST [6263] LOG:  restored log file "000000010000000000000006" from archive
2024-02-15 04:39:55.685 EST [6263] LOG:  redo starts at 0/6000028
2024-02-15 04:39:55.686 EST [6263] LOG:  consistent recovery state reached at 0/6000100
2024-02-15 04:39:55.687 EST [6261] LOG:  database system is ready to accept read-only connections
2024-02-15 04:39:55.696 EST [6263] LOG:  restored log file "000000010000000000000007" from archive
2024-02-15 04:39:55.717 EST [6263] LOG:  restored log file "000000010000000000000008" from archive
2024-02-15 04:39:55.734 EST [6263] LOG:  recovery stopping after WAL location (LSN) "0/8000028"
2024-02-15 04:39:55.734 EST [6263] LOG:  pausing at the end of recovery
2024-02-15 04:39:55.734 EST [6263] HINT:  Execute pg_wal_replay_resume() to promote.
```

Once confirm the database is recovered correctly, we can exit the recovery mode by this psql command:
```
orapg=# \d
               List of relations
 Schema |       Name        | Type  |  Owner
--------+-------------------+-------+----------
 public | document_template | table | postgres
(1 row)

orapg=# select count(*) from document_template;
 count
-------
   400
(1 row)

orapg=#
```
We can see we are able to query the database.

Now lets exit the recovery mode:
```
postgres=# select pg_wal_replay_resume();
 pg_wal_replay_resume
----------------------

(1 row)

postgres=#
```

This command will end the recovery mode and you should be able to insert additional data to the database. The recovery.signal file will be removed, and the future WAL segments will have a new timeline ID.

```
[postgres@pgvm1 data]$ ls -ltr recovery*
ls: cannot access 'recovery*': No such file or directory
[postgres@pgvm1 data]$
```

We will explore more complex scenario in future.