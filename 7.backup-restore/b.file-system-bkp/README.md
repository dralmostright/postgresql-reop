### File system backups 

First, to be technically exact, physical backups are referred to as file system backups in PostgreSQL. As mentioned earlier, this refers to the process of directly copying the directories and files that PostgreSQL uses to store its data, resulting in a complete representation of the database at a specific moment in time. 

An alternative backup strategy is directly copy the files that PostgreSQL uses to store the database in the database.

We can use whatever method we perfer for doing usual file system backups for example:
```
tar -cvf backup.tar /usr/local/pgsql/data
```
The database server must be shutdown or in backup mode in order to get a usuable backup. File system backups only work for complete backup and restoration of an entire database cluster.

In general there are two types of File system backup:
* Offline backups
* Online Backups

#### Offline Backups:
* Taken using OS copy command
* Database server must be shutdown
* Complete Backups 

#### Online Backups:
* Continuous archive must be enabled
* Database Server start/end backup mode
* Complete Backups
* Two methods - Low evel API & pg_basebackup

#### Continuous Archiving:
PostgreSQL maintains WAL files for all transactions in pg_wal directly
PostgreSQL automatically maintains the WAL logs which are full and switched and reused.
Continuous archiving can be setup to keep a copy of switched WAL Logs which can be later used for recovery
It also enables online file system backup of a database cluster.
Requirements:
- wal_level must be set to replica
- archive_mode must be set to on
- archive_command must be set in postgresql.conf which archives WAL logs and supports PITR

#### Continuous Archiving Methods:
##### Archiver Process
* Parameters in postgresql.conf file
	- wal_level=replica
	- archive_mode=on
	- archive_command='cp -i %p /psql/archive/%f'
* Restart the database server
* Archive files are generate after every log switch

Note we can use : 
```
archive_command = 'test ! -f /path/to/database_archive/%f && cp %p /path/to/database_archive/%f'
```
The archive command here first checks to see if the WAL file already exists in the archive, and if it doesn’t, it copies the WAL file to the archive.

#### Streaming WAL
* Parameters in postgresql.conf file
  - wal_level=replica
  - archive_mode=on
  - max_wal_senders=3
* Restart the database server
* pg_receivewal -h localhost -D /psql/archive/
* Transactions are streamed and written to archive files

#### Putting database in archive mode

Firt lets verify the default configuration and their values:
```
postgres=# show wal_level;
 wal_level
-----------
 replica
(1 row)

postgres=# show archive_mode;
 archive_mode
--------------
 off
(1 row)

postgres=# show archive_command;
 archive_command
-----------------
 (disabled)
(1 row)

postgres=#
```

Now lets create the directory to store the archives:
```
[root@pgvm1 ~]# chown -R postgres:postgres /walarc/
[root@pgvm1 ~]#

[postgres@pgvm1 ~]$ mkdir -p /walarc/pg14/archive
```

Now lets make the change and restart the database server:
```
[postgres@pgvm1 data]$ vi postgresql.conf
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop -mf
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA start
waiting for server to start....2024-02-13 01:13:55.686 EST [5992] LOG:  redirecting log output to logging collector process
2024-02-13 01:13:55.686 EST [5992] HINT:  Future log output will appear in directory "log".
 done
server started
[postgres@pgvm1 data]$
```

And now finally verify the change:
```
postgres=# show wal_level;
 wal_level
-----------
 replica
(1 row)

postgres=# show archive_mode;
 archive_mode
--------------
 on
(1 row)

postgres=# show archive_command;
                          archive_command
--------------------------------------------------------------------
 test ! -f /walarc/pg14/archive/%f && cp %p /walarc/pg14/archive/%f
(1 row)

postgres=#
```

### Online Backup using pg_basebackup
* pg_basebackup can be used to take an online base backup of a database cluster
* This backup can be used for PITR or Streaming Replication
* pg_basebackup makes a binary copy of the database cluster files
* System is automatically put in and out of backup mode.

#### Steps require to take Base Backup:
* Modify pg_hba.conf so that the database is accessilbe
* Database must be in contineous archive mode and bew two parameters should also be set on postgresql.conf:
	- max_wal_senders = 3
	- wal_keep_size=512
	
Continuous archive we have already done now lets verify the rest two parameters:
```
postgres=# show max_wal_senders;
 max_wal_senders
-----------------
 10
(1 row)

postgres=# show wal_keep_size;
 wal_keep_size
---------------
 0
(1 row)

postgres=#
```
As we can see ```max_wal_senders``` is already 3+ hence we will make change to ```wal_keep_size```
```
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA stop -mf
waiting for server to shut down.... done
server stopped
[postgres@pgvm1 data]$ vi postgresql.conf
[postgres@pgvm1 data]$
[postgres@pgvm1 data]$ pg_ctl -D $PGDATA start
waiting for server to start....2024-02-13 01:28:56.670 EST [6384] LOG:  redirecting log output to logging collector process
2024-02-13 01:28:56.670 EST [6384] HINT:  Future log output will appear in directory "log".
 done
server started
[postgres@pgvm1 data]$
```

Now lets verify the change:
```
postgres=# show wal_keep_size;
 wal_keep_size
---------------
 512MB
(1 row)

postgres=#
```

### Options for pg_basebackup
| Option	| Description |
| ----------- | ------------- |
|-D <directory name> | Location of backup|
|-F <p or t> | Backup file format. Plain(p) or tar(t)|
|-R | Write standby signal and append postgresql.auto.conf|
|-T OLDDIR=NEWDIR | relocate tablespace in OLDDIR to NEWDIR|
|--waldir | Write ahead logs location|
|-z | enable gzip compression for files|
|-Z level | compression level|
|-P | Progress Reporting|
|-h host | host on which cluster is running|
|-p port | cluster port|

To create a base backup of the server at localhost and store it in the local directory we can use command as below:
pg_basebackup -h localhost -D /...