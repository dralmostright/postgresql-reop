### The Cumulative Statistics System 

PostgreSQL's cumulative statistics system supports collection and reporting of information about server activity. Presently, accesses to tables and indexes in both disk-block and individual-row terms are counted. The total number of rows in each table, and information about vacuum and analyze actions for each table are also counted. If enabled, calls to user-defined functions and the total time spent in each one are counted as well.

PostgreSQL also supports reporting dynamic information about exactly what is going on in the system right now, such as the exact command currently being executed by other server processes, and which other connections exist in the system. This facility is independent of the cumulative statistics system.

#### Statistics Collection Configuration
Since collection of statistics adds some overhead to query execution, the system can be configured to collect or not collect information. This is controlled by configuration parameters that are normally set in postgresql.conf. 

* The parameter track_activities enables monitoring of the current command being executed by any server process.
* The parameter track_counts controls whether cumulative statistics are collected about table and index accesses.
* The parameter track_functions enables tracking of usage of user-defined functions.
* The parameter track_io_timing enables monitoring of block read and write times.
* The parameter track_wal_io_timing enables monitoring of WAL write times.

```
[postgres@pgvm1 ~]$ psql -c 'SHOW track_activities;'
 track_activities
------------------
 on
(1 row)

[postgres@pgvm1 ~]$
```
Normally these parameters are set in postgresql.conf so that they apply to all server processes, but it is possible to turn them on or off in individual sessions using the SET command. 

Cumulative statistics are collected in shared memory. Every PostgreSQL process collects statistics locally, then updates the shared data at appropriate intervals. When a server, including a physical replica, shuts down cleanly, a permanent copy of the statistics data is stored in the pg_stat subdirectory, so that statistics can be retained across server restarts. In contrast, when starting from an unclean shutdown (e.g., after an immediate shutdown, a server crash, starting from a base backup, and point-in-time recovery), all statistics counters are reset.

#### Viewing Statistics
Several predefined views are available to show the current state of the system. There are also several other views available to show the accumulated statistics. Alternatively, one can build custom views using the underlying cumulative statistics functions which we will discuss in next sections.

Cumulative statistics views and functions to monitor collected data are based on certian intervals.

##### Dynamic Statistics Views

| HostName | Instance Type | Operating System & MySQL version |
| ----------- | ----------- | -----------------|

| View Name	| Description |
| ----------- | ------------- |
| pg_stat_activity |One row per server process, showing information related to the current activity of that process, such as state and current query. |
| pg_stat_replication | One row per WAL sender process, showing statistics about replication to that sender's connected standby server.|
| pg_stat_wal_receiver	Only one row, showing statistics about the WAL receiver from that receiver's connected server. See pg_stat_wal_receiver for details.
| pg_stat_recovery_prefetch	Only one row, showing statistics about blocks prefetched during recovery. See pg_stat_recovery_prefetch for details.
| pg_stat_subscription	At least one row per subscription, showing information about the subscription workers. See pg_stat_subscription for details.
| pg_stat_ssl	One row per connection (regular and replication), showing information about SSL used on this connection. See pg_stat_ssl for details.
| pg_stat_gssapi	One row per connection (regular and replication), showing information about GSSAPI authentication and encryption used on this connection. See pg_stat_gssapi for details.
| pg_stat_progress_analyze	One row for each backend (including autovacuum worker processes) running ANALYZE, showing current progress. See Section 28.4.1.
| pg_stat_progress_create_index	One row for each backend running CREATE INDEX or REINDEX, showing current progress. See Section 28.4.4.
| pg_stat_progress_vacuum	One row for each backend (including autovacuum worker processes) running VACUUM, showing current progress. See Section 28.4.5.
| pg_stat_progress_cluster	One row for each backend running CLUSTER or VACUUM FULL, showing current progress. See Section 28.4.2.
| pg_stat_progress_basebackup	One row for each WAL sender process streaming a base backup, showing current progress. See Section 28.4.6.
| pg_stat_progress_copy	One row for each backend running COPY, showing current progress. See Section 28.4.3.