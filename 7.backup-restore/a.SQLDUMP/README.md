### Backup in Postgresql

Postgres comes with a variety of methods to backup and restore the state of your database. It is an essential task for database administrators to ensure data integrity and recover from data loss or system failures.

In this section we will explore logical backup which is SQL dump. The idea behind this dump method is to generate a file with SQL commands that, when fed back to the server, will recreate the database in the same state as it was at the time of the dump. PostgreSQL provides the utility program pg_dump for this purpose and it comes with PostgreSQL it self, it doesn't require special installation.

pg_dump is a regular PostgreSQL client application (albeit a particularly clever one). This means that you can perform this backup procedure from any remote host that has access to the database. But remember that pg_dump does not operate with special permissions. In particular, it must have read access to all tables that you want to back up, so in order to back up the entire database you almost always have to run it as a database superuser.

We can simply take backup by issuing below command:
```
[postgres@testdb ~]$ pg_dump testdb > testdb.sql
[postgres@testdb ~]$ ls -ltr
total 5800
drwx------. 4 postgres postgres    4096 Feb  1 13:16 15
-rw-r--r--. 1 postgres postgres 5931469 Feb  2 02:47 testdb.sql
[postgres@testdb ~]$
```
The pg_dump supports other output formats as well. You can specify the output format using the -F option, where c means custom format archive file, d means directory format archive, and t means tar format archive file: all formats are suitable for input into pg_restore.
```
[postgres@testdb ~]$ pg_dump -F c testdb > testdb.dmp
[postgres@testdb ~]$ pg_dump -F t testdb > testdb.tr
[postgres@testdb ~]$
```

To dump output in the directory output format, use the -f flag (which is used to specify the output file) to specify the target directory instead of a file. The directory which will be created by pg_dump must not exist.
```
[postgres@testdb ~]$ pg_dump -F d testdb -f dumpdir
[postgres@testdb ~]$ ls -ltr dumpdir/
total 3304
-rw-r--r--. 1 postgres postgres    3570 Feb  2 02:52 toc.dat
-rw-r--r--. 1 postgres postgres 3376914 Feb  2 02:52 3351.dat.gz
[postgres@testdb ~]$
```

Postgres comes with a pg_dumpall command which allows the users to backup the whole PostgreSQL cluster. This cluster includes all the databases and roles stored in the Postgres server.
```
[postgres@testdb ~]$ pg_dumpall -U postgres -f backup_file.sql
[postgres@testdb ~]$
```

There are multiple other options available to optimize size and time like increasing number of threads, compressing etc. However here we are exploring on the basics.

### Restore of PostgreSQL

Here again we will discuss only about the logical backup restore.