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