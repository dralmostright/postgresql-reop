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
