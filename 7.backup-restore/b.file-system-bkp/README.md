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