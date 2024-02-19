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


Syllabus of PostGreSQL Admin Online Training CourseModule 1:PostgreSQL Introduction & Architecture
1. Introduction & History
2. PostgreSQL Major Features
3. PostgreSQL Architecture Overview
Module 2: PostgreSQL Data Types
Module 3: PostgreSQL Installation
1. Platforms
2. Binary Installation
3. Source Installation
4. Binary vs. Source – Pros & cons
5. Initializing a PostgreSQL Cluster
6. Starting & Stopping a PostgreSQL Cluster
7. Automatic Startup / Shutdown
8. Common Issues & Troubleshooting
Module 4: PostgreSQL Configuration
1. Access Control
2. The postgresql.conf file
3. Common Issues & Troubleshooting
Module 5: Introduction to psql
1. Command line parameters
2. Meta Commands
3. SET Commands
4. psql Security
Module 6: Functions & Operators
Module 7: Managing PostgreSQL Databases
1. Creating PostgreSQL Databases
2. Creating Schemas
3. Creating Tables
4. Altering Tables
5. SELECT & Joins
6. Indexes & Foreign Keys
Module 8: PostgreSQL Roles and Security
1. Views
2. Rules
3. Users, Groups & Roles
4. Sequences
5. Object Security
Module 9: Moving Data with PostgreSQL
1. Basic DML
2. COPY
3. Other Tools
Module 10: Tablespaces, Inheritance and Data Partitioning
1. Tablespaces
2. Inheritance
3. PostgreSQL Data Partitioning
Module 11: VACUUM
1. Routine Vacuuming
2. Benefits of Vacuuming
3. Recovering Disk Space
4. Updating Planner Statistics
5. Transaction ID Wraparound Failure
6. Vacuum Lab?
Module 12: Transactions & Concurrency Control
1. Transactions
2. Concurrency
Module 13: Routine DBA Tasks and Best Practices
1. Log Management
2. Query Analysis
3. Routine Vacuuming
4. Recovering Disk Space
5. Managing Planner Statistics
6 .REINDEX
7. LAB
Module 14: Monitoring and Statistics
1. Database Logs
2. OS Process Monitoring
3. The PostgreSQL Statistics Collector
4. Statistics Views
5. Statistics Functions
6. LAB
Module 15: PostgreSQL Tools Overview
1. PG Badger
2. PG Bouncer
3. PG Pool
4. PGCLUU
5. PG Admin
6. PG Modeler
7. MySQL Workbench
8. pgbench
9. Consistent State PTS
Module 16: PostgreSQL Performance Tuning
1. OS Tuning
2. HW Configuration
3. Transaction Logs
4. Tablespaces & Partitioning
5. Checkpoint Tuning
6. Query Tuning
Module 17: PostgreSQL Backup and Recovery
1. pg_dump
2. pg_dumpall
3. Recovery Options
4. Restore via a List File
5. Point In Time Recovery (PITR) Based Backup
6. PITR Based Recovery
Module 18: PostgreSQL Upgrade Methods
1 .Minor Version Upgrades
2 .pg_upgrade
3. RPM Based Upgrade
4. Source Based Upgrade
5. SLONY Based Upgrade
Module 19: PostgreSQL Streaming Replication
1. Overview
2. Configuration
3. Base Backup
4 .Recovery.conf
5. Initializing Streaming Replication
6. Standby Conflicts
7. Monitoring
8. Standby Promotion
9. Cascading Replication
10. WAL Shipping
11. Replication Slots
12. Synchronous Replication
Module 20: SLONY
1. Overview
2. Configuration & Setup
3. Monitoring
4. Executing DDL
5. Adding Tables to Replication
6. Switchover
7. Failover
Module 21: PostgreSQL High Availability
1. Overview
2. Replication Type Selection
3. Connection Poolers
4. Heartbeat Monitoring
5. Failing Over
6. Failing Back
Module 22: PostgreSQL and AWS
Module 23: PostgreSQL RDS Overview
Module 24: PostgreSQL Redshift Overview
Module 25: The PostgreSQL Contribs


 Part 3: Installation of PostgreSQL

PostgreSQL Version Types 
Installation on windows
Installation on Linux
Install PostgreSQL in Docker
Installation of PostgreSQL on AWS EC2
AWS PostgreSQL RDS
Using EDB Using Stack Builder
 

Part 4: Database connection

Linux firewall
Ports
Pg_hba.cog
LDAP Auth
OS Authentication
 

Part 4: PostgreSQL Admin tools

pgadmin
Dbeaver
Omnidb
 

Part 3:  Architecture of PostgreSQL Database Engine

Client Process & Application Connection
Server process
Postmaster
Back-end processes   
Background Writer 
Checkpoint
Auto vacuum Launcher
WAL Writer
Stats Collector
Log collector
Archiver
Memory Architecture
Local Memory Area
Work_Mem
Maintenance_Work_mem
Temp_buffers
Shared Memory Area
Shared buffer
Wall Buffer
Commit log
Logical Structure of Database
PG/SQL
Database Cluster
Database
Tables
Indexes 
View 
Function
Sequences
Table spaces
Table Partitioning
Why Partition
Partition types
List partition
Range partition
Hash partition
Multilevel partition 
 

Part 4: Configuration overview

Database files & Directory Layout   
Base
Global
Pg_commit_ts
Pg_clog
pg_logical
Database Config files   
PostgreSQL.conf
pg_hba.conf   
pg_ident.conf   
postgresql.conf   
postgresql.auto.conf
Postmaster.opts
 
Linux environment variables for PostgreSQL
PGHOST 
PGHOSTADDR
PGPORT 
PGDATABASE
PGUSER
PGPASSWORD 
PGPASSFILE
PGSERVICE
PGSERVICEFILE 
PGOPTIONS
 

Part 5: PostgreSQL Database & Objects   

Database creation
Schema creation
User creation
Group creation
 

Part 6: Database Start / Stop

Pg_ctl
Service start/stop Linux
Statue of the database server
Stop fast
Stop immediate
Reload
 

Part 7: Database Catalog Tables

pg_aggregate
pg_am
pg_amop
pg_amproc
pg_attrdef
pg_attribute
pg_authid
pg_auth_members
pg_cast
pg_class
pg_constraint
pg_collation
pg_conversion
pg_database
pg_db_role_setting
pg_default_acl
pg_depend
pg_description
pg_enum
pg_extension
pg_foreign_data_wrapper
pg_foreign_server
pg_foreign_table
pg_index
pg_inherits
pg_language
pg_largeobject
pg_largeobject_metadata
pg_namespace
pg_opclass
pg_operator
pg_opfamily
pg_pltemplate
pg_proc
pg_rewrite
pg_seclabel
pg_shdepend
pg_shdescription
pg_statistic
pg_tablespace
pg_trigger
pg_ts_config
pg_ts_config_map
pg_ts_dict
pg_ts_parser
pg_ts_template
pg_type
pg_user_mapping
System Views
pg_available_extensions
pg_available_extension_versions
pg_cursors
pg_group
pg_indexes
pg_locks
pg_prepared_statements
pg_prepared_xacts
pg_roles
pg_rules
pg_seclabels
pg_settings
pg_shadow
pg_stats
pg_tables
pg_timezone_abbrevs
pg_timezone_names
pg_user
pg_user_mappings
pg_views
 

Part 7:  Managing Storage for PostgreSQL

Table space
Storage volumes
Iscsi
 

Part 8: Backup & Recovery  

Logical backup 
Physical backup
Pg_dump
pg_dumpall
Pg_base backup
Point in time recovery
OS level backup
Tar backup
ISCI level backup
Storage level backup
 

Part 9: PostgreSQL Database Maintenance

Autocaccum
Page Corruption
Auto Freeze
Pg_repack 
Pg_reorg
Preventing transaction ID Wraparound Failures 
Table bloating
Index Bloating 
Planner Stats update
Re-indexing
 

Part 10: Monitoring 

Cpu usage
Page and Swapping 
System load
Disk space monitoring 
Network monitoring
 

Part 11: Monitoring tools installation and configuration

Installation of Prometheus & Grafana 
Prometheus exporters for PostgreSQL
PostgreSQL Enterprise Manager (PEM) & Pem Agent install
Nagios overview for PostgreSQL monitoring
AWS Cloud watch overview
 

Part 12: Data load 

.csv load to tables
Data copy using PostgreSQL_fdw
Copy freeze
 

Part 13: Replication

Types of replications
Advantages of Replication
Replication Configuration
In house to AWS RDS replication 
Hot standby 
Streaming replication
Failover 
Swiftcover 
 

Part 14: Upgrade 

Database version upgrade
Database upgrade 
Database upgrade best particles 
Database upgrade planning
Database upgrade prerequisites 
 

Part 15: Database Connection pooling 

Pg_pool
Pg_bouncer
 

Part 16:  Migration

Database migration using ora2pg
Server migration
PostgreSQL Migration best practices
 

Part 17: Performance Tuning and Optimization

Server parameter tuning & Query tuning  
max_connections           
password_encryption       
Resource Usage           
shared_buffers           
work_mem           
maintenance_work_mem       
effective_io_concurrency   
Write-Ahead Log           
wal_compression           
wal_log_hints           
wal_buffers           
checkpoint_timeout       
checkpoint_completion_target   
max_wal_size           
archive_mode           
archive_command
 

Part 18:  PostgreSQL DBA Daily Tasks & Automation

Backup automation using Shell script
Database log query monitoring
Killing Session using pg_terminate_backend
Application DB performance monitoring
Finding session lock
Running Analyze for latest stats
 

Part 20:  Php or python project on PostgreSQL Database

Database Connection to Application
CURD operation using code
Basic connection configuration and Monitoring 

https://www.koenig-solutions.com/CourseContent/custom/202422620-PostgreSQLDatabaseAdministration.pdf