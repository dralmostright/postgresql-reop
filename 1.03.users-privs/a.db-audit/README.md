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

