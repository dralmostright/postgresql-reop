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

Operating System/PostgreSQL version:
|Operating System | PostgreSQL version |
|-----------------|--------------------|
|OEL 8.5          | PostgreSQL 15      |


Server Configurations:
|Server Type  | IP Address | Port|
|-----------------|--------------------|-----|
|Primary |192.168.229.138 | 5432 |
|Secondary | 192.168.229.134 | 5432 |
