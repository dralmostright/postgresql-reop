### pgPool

One of PostgreSQL open source extensions. It is one of our solutions to meet both requirements: high availability and scalability. pgpool-II is a middleware that can run on Linux and Solaris between applications and databases. Its main features include the following:

|Feature	|Explanation	| Classification|
------------|---------------|-----------|
|Load balancing	|Distributes read-only queries to multiple database servers efficiently, so that more queries can be processed.|	Performance improvement|
|Connection pooling	|Retains/reuses connections to databases to reduce connection overhead and improve performance when reconnection is made.|	Performance improvement|
|Replication	|Replicates data to multiple database servers at any given point in time, for database redundancy.|	High availability of databases|
|Automatic failover|	In case the primary database server fails, automatically switches to the standby server to continue operation.|	High availability of databases|
|Online recovery|	Restores or adds database servers without stopping operation.|	High availability of databases|
|Watchdog	|Links multiple instances of pgpool-II, performs heartbeat monitoring, and shares server information. When failure occurs, a switch is conducted autonomously.|High availability of pgpool-II|
|Limiting Exceeding Connections| There is a limit on the maximum number of concurrent connections with PostgreSQL, and connections are rejected after this many connections. Setting the maximum number of connections, however, increases resource consumption and affect system performance. pgpool-II also has a limit on the maximum number of connections, but extra connections will be queued instead of returning an error immediately.| High availability of databases|

#### Load balancing and connection pooling
Scale-out is one way to increase the processing capacity of the entire database system by adding servers. PostgreSQL allows you to scale out using streaming replication. Efficiently distributing queries from applications to database servers is essential in this scenario. PostgreSQL itself does not have a distribution feature, but you can utilise load balancing of pgpool-II, which efficiently distributes read-only queries in order to balance workload.

Another great pgpool-II feature is connection pooling. By using connection pooling of pgpool-II, connections can be retained and reused, reducing the overhead that occurs when connecting to database servers. To use these features, you need to set the relevant parameters in pgpool.conf.

#### Watchdog
In order to implement high availability of the entire system, pgpool-II itself also needs to be made redundant. This feature for this redundancy is called Watchdog.

Here is how it works. Watchdog links multiple instances of pgpool-II in an active/standby setup. Then, the linked pgpool-II instances perform mutual hearbeat monitoring and share server information (host name, port number, pgpool-II status, virtual IP information, startup time). If pgpool-II (active) providing the service fails, pgpool-II (standby) autonomously detects it and performs failover. When doing this, the new pgpool-II (active) starts a virtual IP interface, and the old pgpool-II (active) stops its virtual IP interface. This allows the application side to use pgpool-II with the same IP address even after the switch of the servers. By using Watchdog, all instances of pgpool-II work together to perform database server monitoring and failover operations - pgpool-II (active) works as the coordinator.

pgpool.conf needs to be set up accordingly for these operations. In a redundant configuration, we also need to take into account the possible consequences of split brain of both pgpool-II and database (PostgreSQL). Split brain is a situation where multiple active servers exist. Updating data during this state will result in data inconsistency and recovery will become onerous. To avoid a split brain, it is recommended to configure pgpool-II with 3 or more servers, and to have an odd number of servers.
