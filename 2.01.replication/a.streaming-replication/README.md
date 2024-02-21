### Replilcation
Streaming replication is a core utility of PostgreSQL introduced in version 9.0.  Streaming replication allows a standby server to stay more up-to-date than is possible with file-based log shipping. The standby connects to the primary, which streams WAL records to the standby as they're generated, without waiting for the WAL file to be filled. PostgreSQL supports two modes of streaming replication: asynchronous and synchronous mode.

Physical replication maintains a full copy of the entire data of a cluster. It uses exact block addresses and employs byte-by-byte replication. In simpler terms, the entire set of data on the primary server is copied to the replica which acts as a standby node.

#### Pros
It is easy to implement since all the database clusters are identical.
It ensures data consistency and high availability at any point since all the replicas hold identical copies of data.
It is Ideal for read-only operations on the replicas.
It’s very efficient since it does not require any special handling.
#### Cons
It is bandwidth-intensive since the entire data is copied and not just small sections of the primary cluster.
It does not offer multi-master database replication.

#### How it works:
When we start the standby instance, it begins by restoring all WAL available in the archive location, calling restore_command, if configured in the standby’s recovery configuration. Once it reaches the end of WAL available there and restore_command fails, the standby tries to restore any WAL available in the pg_wal directory. If that fails, and streaming replication has been configured, the standby tries to connect to the primary server and start streaming WAL from the last valid record found in archive or pg_wal.

Streaming replication requires that the Operating System and PostgreSQL/EPAS (EDB Postgres Advanced Server) versions should be the same across both primary and standby servers. There are a few changes in approach since PostgreSQL version 12 onwards.  

#### Synchronism
* In Asynchronous Replication data is transferred to a different node without waiting for a confirmation of its receiving.
* In Synchronous Replication the data transfer waits - in the case of a COMMIT - for a confirmation of its successful processing on the standby.
Primary parameter: 'synchronous_standby_names' in postgres.conf on master server.

#### Standby Mode

* Hot: In Hot Standby Mode the standby server runs in 'recovery mode', accepts client connections, and processes their read-only queries.
* Warm: In Warm Standby Mode the standby server runs in 'recovery mode' and doesn't allow clients to connect.
* Cold: Although it is not an official PostgreSQL term, Cold Standby Mode can be associated with a not running standby server with log-shipping technique. The WAL files are transferred to the standby but not processed until the standby starts up.

Operating System/PostgreSQL version:
|Operating System | PostgreSQL version |
|-----------------|--------------------|
|OEL 8.5          | PostgreSQL 14.8      |


Server Configurations:
|Server Type  | IP Address | Port|
|-----------------|--------------------|-----|
|Primary |192.168.229.138 | 5432 |
|Secondary | 192.168.229.134 | 5432 |

#### Configuration of Streaming Replication
Now we will go step by step to configure standby:

Configuration changes on Primary:

##### Parameters:

<strong>Changes required in postgresql.conf</strong>
First we will verify and then make changes as needed:
```
postgres=# show listen_addresses;
 listen_addresses
------------------
 *
(1 row)

postgres=# 
postgres=# show archive_mode;
 archive_mode
--------------
 on
(1 row)

postgres=# 
postgres=# show max_wal_senders;
 max_wal_senders
-----------------
 10
(1 row)

postgres=# 
postgres=# show max_wal_size;
 max_wal_size
--------------
 1GB
(1 row)

postgres=# 
postgres=# show wal_level;
 wal_level
-----------
 replica
(1 row)

postgres=# 
postgres=# show hot_standby;
 hot_standby
-------------
 on
(1 row)

postgres=# 
postgres=# show archive_command;
                          archive_command
--------------------------------------------------------------------
 test ! -f /walarc/pg14/archive/%f && cp %p /walarc/pg14/archive/%f
(1 row)

postgres=#
```

As all the required parameters are set, there are no changes required:


<Strong>Changes required in pg_hba.conf</strong>
Set up authentication on the primary server to allow replication connections from the standby server(s).
```
[postgres@pgvm1 data]$ vi pg_hba.conf
[postgres@pgvm1 data]$
[postgres@pgvm1 data]$ cat pg_hba.conf | grep trust
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
host    replication     all             192.168.229.240/32      trust
[postgres@pgvm1 data]$
```

#### Configuration on Standby database instance
We need to install the PostgreSQL binaries of same version, which in my case is already done as i have cloned the whole vm. If thats not the case we need to create appropriate directires and we can do that in two ways :

Option 1
Create a new data directory and set up the ownership and permissions.  In this case, we need to define Postgres Service manually.

Option 2
Initialize the database instance with ```initdb```and remove the entire content from the data directory

After creation of requisite data directory, initiate taking backup using pg_basebackup command (to be executed on the standby server) 
```
[postgres@pgvm2 ~]$ env | grep PGDATA
PGDATA=/var/lib/pgsql/14/data
[postgres@pgvm2 ~]$ pg_basebackup -D $PGDATA -h 192.168.229.138 -p 5432 -Xs -R -P
819504/819504 kB (100%), 1/1 tablespace
[postgres@pgvm2 ~]$ ls -ltr $PGDATA | tail -5
-rw------- 1 postgres postgres     30 Feb 21 12:03 current_logfiles
-rw------- 1 postgres postgres    225 Feb 21 12:03 backup_label.old
-rw------- 1 postgres postgres      0 Feb 21 12:03 standby.signal
-rw------- 1 postgres postgres    350 Feb 21 12:03 postgresql.auto.conf
-rw------- 1 postgres postgres 227804 Feb 21 12:03 backup_manifest
[postgres@pgvm2 ~]$
```
Where the flags define as below:
-D = data directory
-h  = IP address of primary server
-p = Port on which primary instance is running
-Xs = WAL method - stream
-P = Progress information
-R = Write configuration parameters for replication

```primary_conninfo``` will automatically be defined by the pg_basebackup command.  Verify the same in postgresql.auto.conf
```
[postgres@pgvm2 data]$ cat postgresql.auto.conf
# Do not edit this file manually!
# It will be overwritten by the ALTER SYSTEM command.
primary_conninfo = 'user=postgres passfile=''/var/lib/pgsql/.pgpass'' channel_binding=prefer host=192.168.229.138 port=5432 sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any'
[postgres@pgvm2 data]$
```

In case of PostgreSQL version 12 and above, create a blank standby.signal file in data directory location
```
[postgres@pgvm2 data]$ pwd
/var/lib/pgsql/14/data
[postgres@pgvm2 data]$ touch standby.signal
[postgres@pgvm2 data]$
```

Define the below parameters  in postgresql.conf file
```
[postgres@pgvm2 data]$ vi postgresql.conf
[postgres@pgvm2 data]$ cat postgresql.conf | grep restore_command
restore_command = 'rsync -a postgres@192.168.229.138:/walarc/pg14/archive/%f %p'   # command to use to restore an archived logfile segment
[postgres@pgvm2 data]$ cat postgresql.conf | grep recovery_target_timeline
recovery_target_timeline = 'latest'     # 'current', 'latest', or timeline ID
[postgres@pgvm2 data]$
```
With all parameters update, now lets start the postgresql server:
```
[postgres@pgvm2 data]$ pg_ctl start
waiting for server to start....2024-02-21 05:37:38 EST [6416]: [1-1] user=,db=,app=,client= LOG:  redirecting log output to logging collector process
2024-02-21 05:37:38 EST [6416]: [2-1] user=,db=,app=,client= HINT:  Future log output will appear in directory "log".
 done
server started
[postgres@pgvm2 data]$
```

Now lets verify the replication in primary:
```
postgres=# \x
Expanded display is on.
postgres=# select * from pg_stat_replication;
-[ RECORD 1 ]----+------------------------------
pid              | 13513
usesysid         | 10
usename          | postgres
application_name | walreceiver
client_addr      | 192.168.229.140
client_hostname  |
client_port      | 43216
backend_start    | 2024-02-21 05:37:39.796904-05
backend_xmin     |
state            | streaming
sent_lsn         | 0/C3000148
write_lsn        | 0/C3000148
flush_lsn        | 0/C3000148
replay_lsn       | 0/C3000148
write_lag        |
flush_lag        |
replay_lag       |
sync_priority    | 0
sync_state       | async
reply_time       | 2024-02-21 05:41:00.029444-05

postgres=#
```

Now lets verify the replication in standby:
```
postgres=# \x
Expanded display is on.
postgres=# select * from pg_stat_wal_receiver;
-[ RECORD 1 ]---------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
pid                   | 6431
status                | streaming
receive_start_lsn     | 0/C3000000
receive_start_tli     | 2
written_lsn           | 0/C3000148
flushed_lsn           | 0/C3000148
received_tli          | 2
last_msg_send_time    | 2024-02-21 05:42:40.092456-05
last_msg_receipt_time | 2024-02-21 05:42:40.093393-05
latest_end_lsn        | 0/C3000148
latest_end_time       | 2024-02-21 05:37:39.802778-05
slot_name             |
sender_host           | 192.168.229.138
sender_port           | 5432
conninfo              | user=postgres passfile=/var/lib/pgsql/.pgpass channel_binding=prefer dbname=replication host=192.168.229.138 port=5432 fallback_application_name=walreceiver sslmode=prefer sslcompression=0 sslsni=1 ssl_min_protocol_version=TLSv1.2 gssencmode=prefer krbsrvname=postgres target_session_attrs=any

postgres=#
```

#### Doing some testing:

Creating database in primary:
```
postgres=# create database abcdb;
CREATE DATABASE
postgres=# \l abcdb
                              List of databases
 Name  |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges
-------+----------+----------+-------------+-------------+-------------------
 abcdb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(1 row)

postgres=#
```

Checking on Standby:
```
postgres=# \l abcdb
                              List of databases
 Name  |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges
-------+----------+----------+-------------+-------------+-------------------
 abcdb | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
(1 row)

postgres=#
```

#### Checking Replication Lag:
On Primary:
```
postgres=# SELECT pid,application_name,client_addr,client_hostname,state,sync_state,replay_lag FROM pg_stat_replication;
-[ RECORD 1 ]----+----------------
pid              | 13513
application_name | walreceiver
client_addr      | 192.168.229.140
client_hostname  |
state            | streaming
sync_state       | async
replay_lag       |

postgres=#
postgres=# SELECT application_name,
postgres-#     state,
postgres-#     sync_state,
postgres-#     client_addr,
postgres-#     client_hostname,
postgres-#     pg_wal_lsn_diff(pg_current_wal_lsn(),sent_lsn) AS sent_lag,
postgres-#     pg_wal_lsn_diff(sent_lsn,flush_lsn) AS receiving_lag,
postgres-#     pg_wal_lsn_diff(flush_lsn,replay_lsn) AS replay_lag,
postgres-#     pg_wal_lsn_diff(pg_current_wal_lsn(),replay_lsn) AS total_lag,
postgres-#     now()-reply_time AS reply_delay
postgres-# FROM pg_stat_replication
postgres-# ORDER BY client_hostname;
-[ RECORD 1 ]----+----------------
application_name | walreceiver
state            | streaming
sync_state       | async
client_addr      | 192.168.229.140
client_hostname  |
sent_lag         | 0
receiving_lag    | 0
replay_lag       | 0
total_lag        | 0
reply_delay      | 00:00:03.690395

postgres=#
```


On Standby:
```
postgres=# select pg_is_in_recovery(),pg_is_wal_replay_paused(), pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn(), pg_last_xact_replay_timestamp();
-[ RECORD 1 ]-----------------+------------------------------
pg_is_in_recovery             | t
pg_is_wal_replay_paused       | f
pg_last_wal_receive_lsn       | 0/C3002530
pg_last_wal_replay_lsn        | 0/C3002530
pg_last_xact_replay_timestamp | 2024-02-21 05:48:09.272547-05

postgres=#
postgres=# SELECT CASE
postgres-#     WHEN pg_last_wal_receive_lsn() = pg_last_wal_replay_lsn() THEN 0
postgres-#     ELSE EXTRACT(EPOCH FROM now() - pg_last_xact_replay_timestamp())
postgres-# END AS log_delay;
-[ RECORD 1 ]
log_delay | 0

postgres=#
```

#### Automatic removal of obsolete WALs

pg_archivecleanup is used to automatically clean up WAL file archives when running as a standby server. This minimizes the number of WAL files that need to be retained, while preserving crash-restart capability.  The below parameter needs to be included in the postgresql.conf file on the standby server.

This optional parameter specifies a shell command that will be executed at every restartpoint. The purpose of archive_cleanup_command is to provide a mechanism for cleaning up old archived WAL files that are no longer needed by the standby server. Any %r is replaced by the name of the file containing the last valid restart point. That is the earliest file that must be kept to allow a restore to be restartable, and so all files earlier than %r may be safely removed. This information can be used to truncate the archive to just the minimum required to support restart from the current restore. The pgarchivecleanup module is often used in archive_cleanup_command for single-standby configurations, for example:
```archive_cleanup_command = 'pg_archivecleanup /walarc/pg14/archive %r'```

Note however that if multiple standby servers are restoring from the same archive directory, you will need to ensure that you do not delete WAL files until they are no longer needed by any of the servers. archive_cleanup_command would typically be used in a warm-standby configuration (see warm-standby). Write %% to embed an actual % character in the command.

If the command returns a nonzero exit status then a warning log message will be written. An exception is that if the command was terminated by a signal or an error by the shell (such as command not found), a fatal error will be raised.

This parameter can only be set in the postgresql.conf file.
```
postgres=# show archive_cleanup_command;
 archive_cleanup_command
-------------------------

(1 row)

postgres=# 

[postgres@pgvm2 data]$ vi postgresql.conf

postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=#
```

