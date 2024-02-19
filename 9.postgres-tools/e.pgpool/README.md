### pgPool

One of PostgreSQL open source extensions. It is one of our solutions to meet both requirements: high availability and scalability. pgpool-II is a middleware that can run on Linux and Solaris between applications and databases. Its main features include the following:

|Feature	|Classification	|Explanation|
------------|---------------|-----------|
|Load balancing	|Distributes read-only queries to multiple database servers efficiently, so that more queries can be processed.|	Performance improvement|
|Connection pooling	|Retains/reuses connections to databases to reduce connection overhead and improve performance when reconnection is made.|	Performance improvement|
|Replication	|Replicates data to multiple database servers at any given point in time, for database redundancy.|	High availability of databases|
|Automatic failover|	In case the primary database server fails, automatically switches to the standby server to continue operation.|	High availability of databases|
|Online recovery|	Restores or adds database servers without stopping operation.|	High availability of databases|
|Watchdog	|Links multiple instances of pgpool-II, performs heartbeat monitoring, and shares server information. When failure occurs, a switch is conducted autonomously.|High availability of pgpool-II|
|Limiting Exceeding Connections| There is a limit on the maximum number of concurrent connections with PostgreSQL, and connections are rejected after this many connections. Setting the maximum number of connections, however, increases resource consumption and affect system performance. pgpool-II also has a limit on the maximum number of connections, but extra connections will be queued instead of returning an error immediately.| High availability of databases|
