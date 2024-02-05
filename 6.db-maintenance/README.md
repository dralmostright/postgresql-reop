### Database Maintenance Tasks

PostgreSQL, like any database software, requires that certain tasks be performed regularly to achieve optimum performance. The tasks discussed here are required, but they are repetitive in nature and can easily be automated using standard tools such as cron scripts or Windows' Task Scheduler. It is the database administrator's responsibility to set up appropriate scripts, and to check that they execute successfully.

#### What addre the common database maintenance tasks?

Here are the few database maintenance tasks we will be exploring:
* Autovaccum
* Page Corruption
* Auto Freeze
* Pg_repack 
* Pg_reorg
* Preventing transaction ID Wraparound Failures 
* Table bloating
* Index Bloating 
* Planner Stats update
* Re-indexing

#### Autovaccum
Autovacuum is a daemon or background utility process offered by PostgreSQL to users to issue a regular clean-up of redundant data in the database and server. It does not require the user to manually issue the vacuuming and instead, is defined in the postgresql.conf file. 

In a large-scale datacenter, the tables in the database can grow quite large. Performance can degrade significantly if stale and temporary data are not systematically removed. Vacuuming cleans up stale or temporary data in a table, and analyzing refreshes its knowledge of all the tables for the query planner.

PostgreSQL database tables are auto-vacuumed by default when 20% of the rows plus 50 rows are inserted, updated, or deleted. Tables are auto-analyzed when a threshold is met for 10% of the rows plus 50 rows. For example, a table with 10000 rows is not auto-vacuumed until 2050 rows are inserted, updated, or deleted. That same table is auto-analyzed when 1050 rows are inserted, updated, or deleted.

The default auto-vacuum analyze and vacuum settings are sufficient for a small deployment, but the percentage thresholds take longer to trigger as the tables grow larger. Performance degrades significantly before the auto-vacuum vacuuming and analyzing occurs.

It’s important to remember that VACUUM does not rewrite the block and does not "compress" the data. It simply marks the space occupied by dead tuples as "reusable." You can compress data and return the unused space back to the operating system by running VACUUM FULL.
