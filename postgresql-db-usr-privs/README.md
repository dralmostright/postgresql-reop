## Creating Database

PostgreSQL provides two method to create new database which are :
* Using CREATE DATABASE, an SQL command.
* Using createdb a command-line executable.

#### Here we will explore createdb
PostgreSQL command line executable createdb is a wrapper around the SQL command CREATE DATABASE. The only difference between this command and SQL command CREATE DATABASE is that the former can be directly run from the command line and it allows a comment to be added into the database, all in one command.

Verify if the command createdb is reachable and environment variables are properly set
```
[postgres@testdb ~]$ which createdb
/usr/bin/createdb
[postgres@testdb ~]$
```

We can use help to see what options are available and which are mendatory:
```
[postgres@testdb ~]$ createdb --help
createdb creates a PostgreSQL database.

Usage:
  createdb [OPTION]... [DBNAME] [DESCRIPTION]

Options:
  -D, --tablespace=TABLESPACE  default tablespace for the database
  -e, --echo                   show the commands being sent to the server
  -E, --encoding=ENCODING      encoding for the database
  -l, --locale=LOCALE          locale settings for the database
      --lc-collate=LOCALE      LC_COLLATE setting for the database
      --lc-ctype=LOCALE        LC_CTYPE setting for the database
      --icu-locale=LOCALE      ICU locale setting for the database
      --locale-provider={libc|icu}
                               locale provider for the database's default collation
  -O, --owner=OWNER            database user to own the new database
  -S, --strategy=STRATEGY      database creation strategy wal_log or file_copy
  -T, --template=TEMPLATE      template database to copy
  -V, --version                output version information, then exit
  -?, --help                   show this help, then exit

Connection options:
  -h, --host=HOSTNAME          database server host or socket directory
  -p, --port=PORT              database server port
  -U, --username=USERNAME      user name to connect as
  -w, --no-password            never prompt for password
  -W, --password               force password prompt
  --maintenance-db=DBNAME      alternate maintenance database

By default, a database with the same name as the current user is created.

Report bugs to <pgsql-bugs@lists.postgresql.org>.
PostgreSQL home page: <https://www.postgresql.org/>
[postgres@testdb ~]$
```

Now lets create the database
```
[postgres@testdb ~]$ createdb testdb
[postgres@testdb ~]$ 
```

And verify if its created
```
[postgres@testdb ~]$ psql -c "\l"
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
(4 rows)

[postgres@testdb ~]$
```
postgres, template0 and template1 are default database. template* are seed databases, where postgres is medadata database.

Lets test with another option create database:
For this we will need to login to psql prompt and issue create database command as below.
```
[postgres@testdb ~]$ psql postgres
psql (15.5)
Type "help" for help.

postgres=# create database adhikari owner postgres;
CREATE DATABASE
postgres=#
```

Now lets verify if the database has been created.
```
postgres=# \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 adhikari  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 testdb    | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
(5 rows)

postgres=# select datname, oid from pg_database;
  datname  |  oid
-----------+-------
 postgres  |     5
 testdb    | 16398
 template1 |     1
 template0 |     4
 adhikari  | 16399
(5 rows)

postgres=#
```
Every database must have unique name and respective database id will be automatically generated which we can verify physicall in data directory.
```
[postgres@testdb base]$ pwd
/var/lib/pgsql/15/data/base
[postgres@testdb base]$ ls -ltr
total 20
drwx------. 2 postgres postgres 4096 Feb  1 13:31 4
drwx------. 2 postgres postgres 4096 Feb  1 15:36 5
drwx------. 2 postgres postgres 4096 Feb  1 15:36 1
drwx------. 2 postgres postgres 4096 Feb  1 16:08 16398
drwx------. 2 postgres postgres 4096 Feb  1 16:12 16399
[postgres@testdb base]$
```

Now lets test if we are able to login to the database:
```
[postgres@testdb base]$ psql adhikari
psql (15.5)
Type "help" for help.

adhikari=#

[postgres@testdb base]$ psql -h localhost -p 5432 -U postgres testdb
Password for user postgres:
psql (15.5)
Type "help" for help.

testdb=#
```

We can also switch to database using below command:
```
testdb=# \c adhikari
You are now connected to database "adhikari" as user "postgres".
adhikari=#
```

Likewise create database we can also drop database in tow ways 

* Using DROP DATABASE, an SQL command.
* Using dropdb a command-line executable.

Using DROP DATABASE

This command drops a database. It removes the catalog entries for the database and deletes the directory containing the data. It can only be executed by the database owner. This command cannot be executed while you or anyone else is connected to the target database.
```
adhikari=# drop database testdb;
DROP DATABASE
adhikari=#

adhikari=# \l
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 adhikari  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
(4 rows)

adhikari=#
```

Using dropdb Command

PostgresSQL command line executable dropdb is a command-line wrapper around the SQL command DROP DATABASE. There is no effective difference between dropping databases via this utility and via other methods for accessing the server. dropdb destroys an existing PostgreSQL database. The user, who executes this command must be a database super user or the owner of the database.

```
[postgres@testdb base]$ which dropdb
/usr/bin/dropdb
[postgres@testdb base]$ dropdb adhikari;
[postgres@testdb base]$
[postgres@testdb base]$ psql -c "\l"
                                                 List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
-----------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
           |          |          |             |             |            |                 | postgres=CTc/postgres
(3 rows)

[postgres@testdb base]$
````

There are other options which we can pass to dropdb which we can view by --help

We can verify which database we are connected via below command:
```
postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "localhost" (address "::1") at port "5432".
postgres=#
```
