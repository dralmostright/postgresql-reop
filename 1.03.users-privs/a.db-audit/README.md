### Auditing in PostgreSQL
There are two main ways to log SQL:
* Using the PostgreSQL log_statement parameter
* Using the pgaudit extension's pgaudit.log parameter

The log_statement parameter can be set to one of the following options:
* ALL: Logs all SQL statements executed at top level
* MOD: Logs all SQL statements for INSERT, UPDATE, DELETE, and TRUNCATE
* ddl: Logs all SQL statements for DDL commands
* NONE: No statements logged

It is possible to have some statements recorded in the log file but not be visible in the database structure. Most DDL commands in PostgreSQL can be rolled back, so what is in the log is just a list of commands executed by PostgreSQL—not what was actually committed. The log file is not transactional, and it keeps commands that were rolled back. It is possible to display the transaction identifier on each log line by including %x in the log_line_prefix setting, though that has some difficulties in terms of usage.

The pgaudit extension provides two levels of audit logging: session and object levels. The session level has been designed to solve some of the problems of log_statement. pgaudit will log all access, even if it is not executed as a top-level statement, and it will log all dynamic SQL. pgaudit.log can be set to include zero or more of the following settings:
* READ: SELECT and COPY
* WRITE: INSERT, UPDATE, DELETE, TRUNCATE, and COPY
* FUNCTION: Function calls and DO blocks
* ROLE: GRANT, REVOKE, CREATE/ALTER/DROP ROLE
* DDL: All DDL not already included in the ROLE category
* MISC: Miscellaneous—DISCARD, FETCH, CHECKPOINT, VACUUM, and so on

Now lets change the log_statement to ddl, however its always good to verify first:
```
[postgres@testdb ~]$ psql
psql (15.5)
Type "help" for help.

postgres=# show log_statement;
 log_statement
---------------
 none
(1 row)

postgres=#
```

As its set to none lets make the change:
```
[postgres@testdb data]$ vi postgresql.conf
[postgres@testdb data]$ psql
psql (15.5)
Type "help" for help.

postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=#
postgres=# show log_statement;
 log_statement
---------------
 ddl
(1 row)

postgres=#
postgres=# show log_line_prefix;
                 log_line_prefix
-------------------------------------------------
 %t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h
(1 row)

postgres=#
```

Now lets see if DDL statements are being logged or not:
```
postgres=# create database hellodb;
CREATE DATABASE
postgres=#
```

And in log file we can see below entries:
```
2024-02-21 21:58:48 +0545 [1932]: [3-1] user=postgres,db=postgres,app=psql,client=[local] LOG:  statement: create database hellodb;
```
