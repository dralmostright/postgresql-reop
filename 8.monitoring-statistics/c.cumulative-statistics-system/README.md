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


| View Name	| Description |
| ----------- | ------------- |
| pg_stat_activity |One row per server process, showing information related to the current activity of that process, such as state and current query. |
| pg_stat_replication | One row per WAL sender process, showing statistics about replication to that sender's connected standby server.|
| pg_stat_wal_receiver|	Only one row, showing statistics about the WAL receiver from that receiver's connected server. |
| pg_stat_recovery_prefetch |Only one row, showing statistics about blocks prefetched during recovery.|
| pg_stat_subscription| At least one row per subscription, showing information about the subscription workers.|
| pg_stat_ssl |	One row per connection (regular and replication), showing information about SSL used on this connection.|
| pg_stat_gssapi |	One row per connection (regular and replication), showing information about GSSAPI authentication and encryption used on this connection.|
| pg_stat_progress_analyze| One row for each backend (including autovacuum worker processes) running ANALYZE, showing current progress.|
| pg_stat_progress_create_index	| One row for each backend running CREATE INDEX or REINDEX, showing current progress. |
| pg_stat_progress_vacuum |	One row for each backend (including autovacuum worker processes) running VACUUM, showing current progress.|
| pg_stat_progress_cluster |	One row for each backend running CLUSTER or VACUUM FULL, showing current progress. |
| pg_stat_progress_basebackup	| One row for each WAL sender process streaming a base backup, showing current progress.|
| pg_stat_progress_copy	| One row for each backend running COPY, showing current progress. |

##### Accumulated statistics or Collected Statistics Views
<table class="table" summary="Collected Statistics Views" border="1">
          <thead>
            <tr>
              <th>View Name</th>
              <th>Description</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><code class="structname">pg_stat_archiver</code><a id="id-1.6.15.7.6.8.2.2.1.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.1.1.2"></a></td>
              <td>One row only, showing statistics about the WAL archiver process's activity. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-ARCHIVER-VIEW" title="28.2.12.&nbsp;pg_stat_archiver"><code class="structname">pg_stat_archiver</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_bgwriter</code><a id="id-1.6.15.7.6.8.2.2.2.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.2.1.2"></a></td>
              <td>One row only, showing statistics about the background writer process's activity. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-BGWRITER-VIEW" title="28.2.14.&nbsp;pg_stat_bgwriter"><code class="structname">pg_stat_bgwriter</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_database</code><a id="id-1.6.15.7.6.8.2.2.3.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.3.1.2"></a></td>
              <td>One row per database, showing database-wide statistics. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-DATABASE-VIEW" title="28.2.16.&nbsp;pg_stat_database"><code class="structname">pg_stat_database</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_database_conflicts</code><a id="id-1.6.15.7.6.8.2.2.4.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.4.1.2"></a></td>
              <td>One row per database, showing database-wide statistics about query cancels due to conflict with recovery on standby servers. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-DATABASE-CONFLICTS-VIEW" title="28.2.17.&nbsp;pg_stat_database_conflicts"><code class="structname">pg_stat_database_conflicts</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_io</code><a id="id-1.6.15.7.6.8.2.2.5.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.5.1.2"></a></td>
              <td>One row for each combination of backend type, context, and target object containing cluster-wide I/O statistics. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-IO-VIEW" title="28.2.13.&nbsp;pg_stat_io"><code class="structname">pg_stat_io</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_replication_slots</code><a id="id-1.6.15.7.6.8.2.2.6.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.6.1.2"></a></td>
              <td>One row per replication slot, showing statistics about the replication slot's usage. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-REPLICATION-SLOTS-VIEW" title="28.2.5.&nbsp;pg_stat_replication_slots"><code class="structname">pg_stat_replication_slots</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_slru</code><a id="id-1.6.15.7.6.8.2.2.7.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.7.1.2"></a></td>
              <td>One row per SLRU, showing statistics of operations. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-SLRU-VIEW" title="28.2.24.&nbsp;pg_stat_slru"><code class="structname">pg_stat_slru</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_subscription_stats</code><a id="id-1.6.15.7.6.8.2.2.8.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.8.1.2"></a></td>
              <td>One row per subscription, showing statistics about errors. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-SUBSCRIPTION-STATS" title="28.2.9.&nbsp;pg_stat_subscription_stats"><code class="structname">pg_stat_subscription_stats</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_wal</code><a id="id-1.6.15.7.6.8.2.2.9.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.9.1.2"></a></td>
              <td>One row only, showing statistics about WAL activity. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-WAL-VIEW" title="28.2.15.&nbsp;pg_stat_wal"><code class="structname">pg_stat_wal</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_all_tables</code><a id="id-1.6.15.7.6.8.2.2.10.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.10.1.2"></a></td>
              <td>One row for each table in the current database, showing statistics about accesses to that specific table. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-ALL-TABLES-VIEW" title="28.2.18.&nbsp;pg_stat_all_tables"><code class="structname">pg_stat_all_tables</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_sys_tables</code><a id="id-1.6.15.7.6.8.2.2.11.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.11.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_all_tables</code>, except that only system tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_user_tables</code><a id="id-1.6.15.7.6.8.2.2.12.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.12.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_all_tables</code>, except that only user tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_xact_all_tables</code><a id="id-1.6.15.7.6.8.2.2.13.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.13.1.2"></a></td>
              <td>Similar to <code class="structname">pg_stat_all_tables</code>, but counts actions taken so far within the current transaction (which are <span class="emphasis"><em>not</em></span> yet included in <code class="structname">pg_stat_all_tables</code> and related views). The columns for numbers of live and dead rows and vacuum and analyze actions are not present in this view.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_xact_sys_tables</code><a id="id-1.6.15.7.6.8.2.2.14.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.14.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_xact_all_tables</code>, except that only system tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_xact_user_tables</code><a id="id-1.6.15.7.6.8.2.2.15.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.15.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_xact_all_tables</code>, except that only user tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_all_indexes</code><a id="id-1.6.15.7.6.8.2.2.16.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.16.1.2"></a></td>
              <td>One row for each index in the current database, showing statistics about accesses to that specific index. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-ALL-INDEXES-VIEW" title="28.2.19.&nbsp;pg_stat_all_indexes"><code class="structname">pg_stat_all_indexes</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_sys_indexes</code><a id="id-1.6.15.7.6.8.2.2.17.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.17.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_all_indexes</code>, except that only indexes on system tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_user_indexes</code><a id="id-1.6.15.7.6.8.2.2.18.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.18.1.2"></a></td>
              <td>Same as <code class="structname">pg_stat_all_indexes</code>, except that only indexes on user tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_user_functions</code><a id="id-1.6.15.7.6.8.2.2.19.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.19.1.2"></a></td>
              <td>One row for each tracked function, showing statistics about executions of that function. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STAT-USER-FUNCTIONS-VIEW" title="28.2.23.&nbsp;pg_stat_user_functions"><code class="structname">pg_stat_user_functions</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_stat_xact_user_functions</code><a id="id-1.6.15.7.6.8.2.2.20.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.20.1.2"></a></td>
              <td>Similar to <code class="structname">pg_stat_user_functions</code>, but counts only calls during the current transaction (which are <span class="emphasis"><em>not</em></span> yet included in <code class="structname">pg_stat_user_functions</code>).</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_all_tables</code><a id="id-1.6.15.7.6.8.2.2.21.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.21.1.2"></a></td>
              <td>One row for each table in the current database, showing statistics about I/O on that specific table. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STATIO-ALL-TABLES-VIEW" title="28.2.20.&nbsp;pg_statio_all_tables"><code class="structname">pg_statio_all_tables</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_sys_tables</code><a id="id-1.6.15.7.6.8.2.2.22.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.22.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_tables</code>, except that only system tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_user_tables</code><a id="id-1.6.15.7.6.8.2.2.23.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.23.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_tables</code>, except that only user tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_all_indexes</code><a id="id-1.6.15.7.6.8.2.2.24.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.24.1.2"></a></td>
              <td>One row for each index in the current database, showing statistics about I/O on that specific index. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STATIO-ALL-INDEXES-VIEW" title="28.2.21.&nbsp;pg_statio_all_indexes"><code class="structname">pg_statio_all_indexes</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_sys_indexes</code><a id="id-1.6.15.7.6.8.2.2.25.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.25.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_indexes</code>, except that only indexes on system tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_user_indexes</code><a id="id-1.6.15.7.6.8.2.2.26.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.26.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_indexes</code>, except that only indexes on user tables are shown.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_all_sequences</code><a id="id-1.6.15.7.6.8.2.2.27.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.27.1.2"></a></td>
              <td>One row for each sequence in the current database, showing statistics about I/O on that specific sequence. See <a class="link" href="monitoring-stats.html#MONITORING-PG-STATIO-ALL-SEQUENCES-VIEW" title="28.2.22.&nbsp;pg_statio_all_sequences"><code class="structname">pg_statio_all_sequences</code></a> for details.</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_sys_sequences</code><a id="id-1.6.15.7.6.8.2.2.28.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.28.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_sequences</code>, except that only system sequences are shown. (Presently, no system sequences are defined, so this view is always empty.)</td>
            </tr>
            <tr>
              <td><code class="structname">pg_statio_user_sequences</code><a id="id-1.6.15.7.6.8.2.2.29.1.2" class="indexterm" name="id-1.6.15.7.6.8.2.2.29.1.2"></a></td>
              <td>Same as <code class="structname">pg_statio_all_sequences</code>, except that only user sequences are shown.</td>
            </tr>
          </tbody>
</table>

#### Wait events:

Understanding PostgreSQL Wait Statistics is critical for optimizing database performance. Wait Statistics provide insights into the resources that are being blocked and the time that is being spent waiting for those resources. Here’s an overview of PostgreSQL Wait Events:

<table><tbody><tr><th>Wait Event Name</th><th>Description</th><th>Impact on Performance</th></tr><tr><td>LWLock</td><td>A lightweight lock that is used to synchronize access to shared memory data structures</td><td>Can cause contention on hot shared memory data structures</td></tr><tr><td>Lock</td><td>A lock that is used to synchronize access to a relation, page, or tuple</td><td>Can cause contention on frequently accessed tables or indexes</td></tr><tr><td>Buffer Pin</td><td>Waiting for a buffer to be pinned in memory</td><td>Can indicate a shortage of buffer cache memory</td></tr><tr><td>Buffer IO</td><td>Waiting for an I/O operation to complete on a buffer</td><td>Can indicate slow disk I/O</td></tr><tr><td>Extension</td><td>Waiting for a shared library extension to load or unload</td><td>Can indicate issues with shared library configuration</td></tr><tr><td>IPC</td><td>Waiting for an interprocess communication resource</td><td>Can indicate contention for system resources</td></tr><tr><td>Lock Manager</td><td>Waiting for the lock manager to complete a request</td><td>Can indicate contention for lock manager resources</td></tr><tr><td>Authentication</td><td>Waiting for a user authentication request to complete</td><td>Can indicate slow user authentication</td></tr><tr><td>Replication Sender</td><td>Waiting for the replication sender to catch up with the standby</td><td>Can indicate issues with replication lag</td></tr></tbody></table>