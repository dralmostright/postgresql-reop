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