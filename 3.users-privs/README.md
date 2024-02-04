## User/Schema/Role/Privileges in PostgreSQL

While thinking of user, role and privileges in PostgreSQL its same like in other RDBMS e.g. Oracle, Mysql. Like other access control mechanisms, PostgreSQL’s access control can be explained like "Role X is allowed to do Y on object Z".

Roles are basically users and groups. It acts like both; you can log in as a role, and a role can belong to another role. Each role has an attribute like LOGIN and INHERIT that indicate whether you can log in as that role and whether the role inherits privileges from the roles it belongs to. You can add a role to a member of another role by using GRANT ROLE ... command. There is only one type of authentication principal in PostgreSQL, a ROLE, which exists at the cluster level. By convention, a ROLE that allows login is considered a user, while a role that is not allowed to login is a group. Please note, while the CREATE USER and CREATE GROUP commands still exist, they are simply aliases for CREATE ROLE.

Objects in PostgreSQL are databases, tables, etc.. There is a tree structure in PostgreSQL objects. A PostgreSQL instance can have multiple databases. A database can have multiple schemas. A schema can have multiple tables. Anything that can be created or accessed in the PostgreSQL cluster is referred to as an object. Databases, schema, tables, views, procedures, functions, and more can each have different privileges applied to them for any role.

Privileges are permissions defined over PostgreSQL objects. For example, there is a SELECT privilege on tables, which is a permission to run SELECT queries on them. Every kind of object has a different set of privileges.

### Roles

Recall that in PostgreSQL both users and groups are technically roles. These are always created at the cluster level and granted privileges to databases and other objects therein. For the purposes of this article, all example user roles will be created with password authentication. Other authentication methods are available, including GSSPI, SSPI, Kerberos, Certificate, and others. However, setting up these alternative methods is beyond what we need to discuss object ownership and privileges.

<strong>Create a User Role</strong>
To create a user role in PostgreSQL, execute the following DDL as a user that has the 'CREATEROLE' privilege.
```
postgres=# CREATE ROLE testusr1 WITH LOGIN PASSWORD 'welcome1';
CREATE ROLE
postgres=#
```

To list the users we can type \du from shell. 
```
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 testusr1  |                                                            | {}

postgres=#
```

Additionally we can query database level too, to find the list of users in the cluster.
```
postgres=# select * from pg_catalog.pg_user;
 usename  | usesysid | usecreatedb | usesuper | userepl | usebypassrls |  passwd  | valuntil | useconfig
----------+----------+-------------+----------+---------+--------------+----------+----------+-----------
 postgres |       10 | t           | t        | t       | t            | ******** |          |
 testusr1 |    16385 | f           | f        | f       | f            | ******** |          |
(2 rows)

postgres=#
```

Alternatively, PostgreSQL still supports the older CREATE USER command, but it's just an alias for CREATE ROLE. In theory it will be deprecated at some point, so users should tend towards CREATE ROLE.

This is the most basic needed for role/user at this point, we will explore more as we go.

### Group Role

To create a group role in PostgreSQL, create a role that is not allowed to login. As mentioned earlier, this is simply a convention that denotes the role as a group.
```
postgres=# CREATE ROLE testgrp WITH NOLOGIN;
CREATE ROLE
postgres=# \du
                                   List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 testgrp   | Cannot login                                               | {}
 testusr1  |                                                            | {}

postgres=#
```
Like user roles, PostgreSQL still supports the older CREATE GROUP command, although it is a direct alias for CREATE ROLE because all roles are created with NOLOGIN by default, which as we've discussed, means the role is used as a group. There is no advantage of using CREATE GROUP and it may be deprecated at some point. 

There are numerous other role attributes that can be applied at the time of creation or through ALTER ROLE. The below snapshot is from PostgreSQL documentation.

```
CREATE ROLE name [ [ WITH ] option [ ... ] ]

where option can be:

      SUPERUSER | NOSUPERUSER
    | CREATEDB | NOCREATEDB
    | CREATEROLE | NOCREATEROLE
    | INHERIT | NOINHERIT
    | LOGIN | NOLOGIN
    | REPLICATION | NOREPLICATION
    | BYPASSRLS | NOBYPASSRLS
    | CONNECTION LIMIT connlimit
    | [ ENCRYPTED ] PASSWORD 'password' | PASSWORD NULL
    | VALID UNTIL 'timestamp'
    | IN ROLE role_name [, ...]
    | IN GROUP role_name [, ...]
    | ROLE role_name [, ...]
    | ADMIN role_name [, ...]
    | USER role_name [, ...]
    | SYSID uid
```

### PUBLIC Role
Every PostgreSQL cluster has another implicit role called PUBLIC which cannot be deleted. All other roles are always granted membership in PUBLIC by default and inherit whatever privileges are currently assigned to it. Up to the latest release PUBLIC role has by default CONNECT, TEMPORARY, USAGE and EXECUTE privileges.

The main thing to notice here is that the PUBLIC role always has the CONNECT privilege granted by default, which conveniently allows all roles to connect to a newly created database. Without the privilege to connect to a database, none of our newly created roles would be able to do much.

### Schemas
In PostgreSQL, a schema is a namespace that contains named database objects such as tables, views, indexes, data types, functions, stored procedures and operators.

A database contains one or more named schemas, which in turn contain tables. Schemas also contain other kinds of named objects, including data types, functions, and operators. The same object name can be used in different schemas without conflict; for example, both schema1 and myschema can contain tables named mytable. Unlike databases, schemas are not rigidly separated: a user can access objects in any of the schemas in the database they are connected to, if they have privileges to do so.

There are several reasons why one might want to use schemas:

* To allow many users to use one database without interfering with each other.
* To organize database objects into logical groups to make them more manageable.
* Third-party applications can be put into separate schemas so they do not collide with the names of other objects.

Schemas are analogous to directories at the operating system level, except that schemas cannot be nested.

To create a schema, use the CREATE SCHEMA command. Give the schema a name of your choice. For example:
```
orapg=# \c orapg
You are now connected to database "orapg" as user "postgres".
orapg=# create schema hr;
CREATE SCHEMA
orapg=#
```

To create or access objects in a schema, write a qualified name consisting of the schema name and table name separated by a dot:

Actually, the even more general syntax ```database.schema.table``` can be used too, but at present this is just for pro forma compliance with the SQL standard. If you write a database name, it must be the same as the database you are connected to.

#### The public schema
PostgreSQL automatically creates a schema called public for every new database. Whatever object you create without specifying the schema name, PostgreSQL will place it into this public schema. Therefore, the following statements are equivalent:
```
CREATE TABLE table_name(
  ...
);
```
and
```
CREATE TABLE public.table_name(
   ...
);
```