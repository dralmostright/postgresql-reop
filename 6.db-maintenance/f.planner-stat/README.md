### Planner Statistics

You'll often hear the term planner statistics thrown around by database geeks. Did you update your statistics. This lingo isn't even limited to PostgreSQL, but is part and parcel to how most decent databases work. For example in PostgreSQL you do a vacuum analyze to update your planner statistics in addition to cleaning up dead space. In SQL Server you do an UPDATE STATISTICS. In MySQL you do an ANALYZE TABLE or a more invasive OPTIMIZE TABLE.

Normally all this "update your stats so your planner can be happy" is usually unnecessary unless you just did a bulk load or a bulk delete or you are noticing your queries are suddenly slowing down. These stat things are generally updated behind the scenes by most databases on an as needed basis.

What makes SQL really interesting and a bit different from procedural languages is that it is declarative (like functional and logical programming languages) and relies on the database planner to come up with strategies for navigating the data. Its strategy is not fixed as it is in procedural languages. A big part of this strategy is decided on by the query planner which looks at distributions of data. Given different WHERE conditions for similar queries, it could come up with vastly different strategies if one value has a significantly higher distribution in a table than another. This is also the mystery of why it sometimes refuses to use an index on a field because it has decided a table scan is more efficient and also why some people consider HINTS evil because they pollute the imperative nature of the language.

We can use pg_stats view to view the stats of a table:
```
testdb=# SELECT attname As colname, n_distinct,
array_to_string(most_common_vals, E'\n') AS common_vals,
array_to_string(most_common_freqs, E'\n') As dist_freq
FROM pg_stats
where tablename='burritos';
 colname  | n_distinct | common_vals |   dist_freq
----------+------------+-------------+---------------
 id       |         -1 |             |
 title    |         -1 |             |
 toppings |         -1 |             |
 thoughts |         -1 |             |
 code     |   -0.43404 | c6aa       +| 0.0002       +
          |            | 633d       +| 0.00016666666+
          |            | 66d5       +| 0.00016666666+
          |            | 6f70       +| 0.00016666666+
          |            | 838d       +| 0.00016666666+
          |            | 8a46        | 0.00016666666
(5 rows)

testdb=#
```

Most queries retrieve only a fraction of the rows in a table, due to WHERE clauses that restrict the rows to be examined. The planner thus needs to make an estimate of the selectivity of WHERE clauses, that is, the fraction of rows that match each condition in the WHERE clause. The information used for this task is stored in the pg_statistic system catalog. Entries in pg_statistic are updated by the ANALYZE and VACUUM ANALYZE commands, and are always approximate even when freshly updated.

Rather than look at pg_statistic directly, it's better to look at its view pg_stats when examining the statistics manually. pg_stats is designed to be more easily readable. Furthermore, pg_stats is readable by all, whereas pg_statistic is only readable by a superuser.

### Extended Statistics 
It is common to see slow queries running bad execution plans because multiple columns used in the query clauses are correlated. The planner normally assumes that multiple conditions are independent of each other, an assumption that does not hold when column values are correlated. Regular statistics, because of their per-individual-column nature, cannot capture any knowledge about cross-column correlation. However, PostgreSQL has the ability to compute multivariate statistics, which can capture such information.

Because the number of possible column combinations is very large, it's impractical to compute multivariate statistics automatically. Instead, extended statistics objects, more often called just statistics objects, can be created to instruct the server to obtain statistics across interesting sets of columns.

Statistics objects are created using the CREATE STATISTICS command. Creation of such an object merely creates a catalog entry expressing interest in the statistics. Actual data collection is performed by ANALYZE (either a manual command, or background auto-analyze). The collected values can be examined in the pg_statistic_ext_data catalog.

ANALYZE computes extended statistics based on the same sample of table rows that it takes for computing regular single-column statistics. Since the sample size is increased by increasing the statistics target for the table or any of its columns (as described in the previous section), a larger statistics target will normally result in more accurate extended statistics, as well as more time spent calculating them.
