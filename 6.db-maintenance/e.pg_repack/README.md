### pg_repack 

pg_repackt is a PostgreSQL extension which lets you remove bloat from tables and indexes, and optionally restore the physical order of clustered indexes. Unlike CLUSTER and VACUUM FULL it works online, without holding an exclusive lock on the processed tables during processing. pg_repack is a fork of the previous pg_reorg project.

Requirements:

* Only superusers can use the utility.
* Target table must have a PRIMARY KEY, or at least a UNIQUE total index on a NOT NULL column.
* PostgreSQL >= 9.4, 9.5, 9.6, 10, 11, 12, 13, 14, 15

Lets try to create the extension:
```
testdb=# create extension pg_repack;
ERROR:  extension "pg_repack" is not available
DETAIL:  Could not open extension control file "/usr/pgsql-15/share/extension/pg_repack.control": No such file or directory.
HINT:  The extension must first be installed on the system where PostgreSQL is running.
testdb=#
```
Seems the extenstion is not present in your system, hence we might need to install it.


Tho install the pg_repack we need to ensure we have installed dependent packages like ```gcc```,```make```,```postgresql-devel``` and other dependencies applicate. Now lets proceed to installation:

Lets first download the [pg_repack](https://pgxn.org/dist/pg_repack/) and unzip it. I am using curl, but we can use any other options like wget or any applicable.
```
[postgres@testdb ~]$ curl -k -O https://api.pgxn.org/dist/pg_repack/1.5.0/pg_repack-1.5.0.zip
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  125k  100  125k    0     0  65552      0  0:00:01  0:00:01 --:--:-- 65552
[postgres@testdb ~]$ 
[postgres@testdb ~]$ ls -ltr pg_repack-1.5.0.zip
-rw-r--r--. 1 postgres postgres 128418 Feb  7 17:20 pg_repack-1.5.0.zip
[postgres@testdb ~]$ 
[postgres@testdb ~]$ unzip pg_repack-1.5.0.zip
Archive:  pg_repack-1.5.0.zip
78421dcc30a59fb9fd9d7bfeda97009f7050f112
   creating: pg_repack-1.5.0/
   creating: pg_repack-1.5.0/.github/
   creating: pg_repack-1.5.0/.github/workflows/
  inflating: pg_repack-1.5.0/.github/workflows/regression.yml
  inflating: pg_repack-1.5.0/.gitignore
  inflating: pg_repack-1.5.0/COPYRIGHT
.....
.....
.....
  inflating: pg_repack-1.5.0/regress/sql/repack-run.sql
  inflating: pg_repack-1.5.0/regress/sql/repack-setup.sql
  inflating: pg_repack-1.5.0/regress/sql/tablespace.sql
  inflating: pg_repack-1.5.0/regress/sql/trigger.sql
  inflating: pg_repack-1.5.0/regress/travis_prepare.sh
  inflating: pg_repack-1.5.0/regress/travis_test.sh
[postgres@testdb ~]$
```

```
[postgres@testdb ~]$ cd pg_repack-1.5.0[postgres@testdb ~]$ cd pg_repack-1.5.0
[postgres@testdb pg_repack-1.5.0]$ make
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -I/usr/pgsql-15/include -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/pgsql-15/include/server -I/usr/pgsql-15/include/internal  -D_GNU_SOURCE -I/usr/include/libxml2  -I/usr/include  -c -o pg_repack.o pg_repack.c
gcc: error: /usr/lib/rpm/redhat/redhat-hardened-cc1: No such file or directory
make[1]: *** [<builtin>: pg_repack.o] Error 1
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
make: *** [Makefile:35: all] Error 2
[postgres@testdb pg_repack-1.5.0]$
```

If you get similar error, you need to install ```redhat-rpm-config```
```
[root@testdb ~]# yum install redhat-rpm-config
Last metadata expiration check: 0:19:30 ago on Wed 07 Feb 2024 05:04:53 PM +0545.
Dependencies resolved.
==============================================================================================================
 Package                       Architecture    Version                        Repository                  Size
==============================================================================================================
Installing:
 redhat-rpm-config             noarch          131-1.0.1.el8                  ol8_appstream               91 k
Upgrading:
 glibc                         x86_64          2.28-236.0.1.el8.7             ol8_baseos_latest          2.2 M
...
...
```

And during installation if you encounter issues like below :
```
/usr/bin/ld: cannot find -lzstd
```

You need to check for the libfile and in case if its not found you might need to install the respective packages
```
[root@testdb pg_repack-1.5.0]# ld -lzstd -verbose
GNU ld version 2.30-123.0.1.el8
  Supported emulations:
   elf_x86_64
   elf32_x86_64
.......
......
......


==================================================
attempt to open //usr/x86_64-redhat-linux/lib64/libzstd.so failed
attempt to open //usr/x86_64-redhat-linux/lib64/libzstd.a failed
attempt to open //usr/lib64/libzstd.so failed
attempt to open //usr/lib64/libzstd.a failed
attempt to open //usr/local/lib64/libzstd.so failed
attempt to open //usr/local/lib64/libzstd.a failed
attempt to open //lib64/libzstd.so failed
attempt to open //lib64/libzstd.a failed
attempt to open //usr/x86_64-redhat-linux/lib/libzstd.so failed
attempt to open //usr/x86_64-redhat-linux/lib/libzstd.a failed
attempt to open //usr/local/lib/libzstd.so failed
attempt to open //usr/local/lib/libzstd.a failed
attempt to open //lib/libzstd.so failed
attempt to open //lib/libzstd.a failed
attempt to open //usr/lib/libzstd.so failed
attempt to open //usr/lib/libzstd.a failed
ld: cannot find -lzstd
[root@testdb pg_repack-1.5.0]#
```

After fixing all the hiccups i was finally able to make clean make. Please don't forget to install respective rpms:
```
[root@testdb ~]# ls -ltr | grep post | grep rpm
-rw-r--r--. 1 root root  156303 Feb  7 21:42 postgresql-static-15.5-1.module+el8.9.0+90110+d8a562d5.x86_64.rpm
-rw-r--r--. 1 root root 1428115 Feb  7 21:44 postgresql-server-devel-15.5-1.module+el8.9.0+90110+d8a562d5.x86_64.rpm
-rw-r--r--. 1 root root  134867 Feb  7 21:50 postgresql-private-libs-15.5-1.module+el8.9.0+90110+d8a562d5.x86_64.rpm
-rw-r--r--. 1 root root   65339 Feb  7 21:54 postgresql-private-devel-15.5-1.module+el8.9.0+90110+d8a562d5.x86_64.rpm
[root@testdb ~]#
```

```
[postgres@testdb pg_repack-1.5.0]$ make
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -I/usr/include -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/include/pgsql/server -I/usr/include/pgsql/internal  -D_GNU_SOURCE -I/usr/include/libxml2   -c -o pg_repack.o pg_repack.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -I/usr/include -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/include/pgsql/server -I/usr/include/pgsql/internal  -D_GNU_SOURCE -I/usr/include/libxml2   -c -o pgut/pgut.o pgut/pgut.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -I/usr/include -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/include/pgsql/server -I/usr/include/pgsql/internal  -D_GNU_SOURCE -I/usr/include/libxml2   -c -o pgut/pgut-fe.o pgut/pgut-fe.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection pg_repack.o pgut/pgut.o pgut/pgut-fe.o  -L/usr/lib64 -Wl,-z,relro  -Wl,-z,now -specs=/usr/lib/rpm/redhat/redhat-hardened-ld  -Wl,--as-needed  -L/usr/lib64 -lpq -L/usr/lib64/pgsql -lpgcommon -lpgport -lssl -lcrypto -lz -lreadline -lpthread -lrt -ldl -lm -o pg_repack
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/lib'
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fPIC -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/include/pgsql/server -I/usr/include/pgsql/internal  -D_GNU_SOURCE -I/usr/include/libxml2   -c -o repack.o repack.c
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fPIC -DREPACK_VERSION=1.5.0 -I. -I./ -I/usr/include/pgsql/server -I/usr/include/pgsql/internal  -D_GNU_SOURCE -I/usr/include/libxml2   -c -o pgut/pgut-spi.o pgut/pgut-spi.c
( echo '{ global:'; gawk '/^[^#]/ {printf "%s;\n",$1}' exports.txt; echo ' local: *; };' ) >exports.list
gcc -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Wendif-labels -Wmissing-format-attribute -Wimplicit-fallthrough=3 -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -fexcess-precision=standard -Wno-format-truncation -Wno-stringop-truncation -O2 -g -pipe -Wall -Werror=format-security -Wp,-D_FORTIFY_SOURCE=2 -Wp,-D_GLIBCXX_ASSERTIONS -fexceptions -fstack-protector-strong -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -specs=/usr/lib/rpm/redhat/redhat-annobin-cc1 -m64 -mtune=generic -fasynchronous-unwind-tables -fstack-clash-protection -fcf-protection -fPIC -shared -Wl,--version-script=exports.list -o pg_repack.so repack.o pgut/pgut-spi.o -L/usr/lib64  -Wl,-z,relro  -Wl,-z,now -specs=/usr/lib/rpm/redhat/redhat-hardened-ld  -Wl,--as-needed
sed 's,REPACK_VERSION,1.5.0,g' pg_repack.sql.in \
| sed 's,relhasoids,false,g'> pg_repack--1.5.0.sql;
sed 's,REPACK_VERSION,1.5.0,g' pg_repack.control.in > pg_repack.control
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/lib'
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/regress'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/regress'
[postgres@testdb pg_repack-1.5.0]$

```

Finally run make install
```
[postgres@testdb pg_repack-1.5.0]$ sudo make install

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for postgres:
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
/usr/bin/mkdir -p '/usr/bin'
/usr/bin/install -c  pg_repack '/usr/bin'
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/bin'
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/lib'
/usr/bin/mkdir -p '/usr/lib64/pgsql'
/usr/bin/mkdir -p '/usr/share/pgsql/extension'
/usr/bin/mkdir -p '/usr/share/pgsql/extension'
/usr/bin/install -c -m 755  pg_repack.so '/usr/lib64/pgsql/pg_repack.so'
/usr/bin/install -c -m 644 .//pg_repack.control '/usr/share/pgsql/extension/'
/usr/bin/install -c -m 644  pg_repack--1.5.0.sql pg_repack.control '/usr/share/pgsql/extension/'
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/lib'
make[1]: Entering directory '/var/lib/pgsql/pg_repack-1.5.0/regress'
make[1]: Nothing to be done for 'install'.
make[1]: Leaving directory '/var/lib/pgsql/pg_repack-1.5.0/regress'
[postgres@testdb pg_repack-1.5.0]$

[postgres@testdb ~]$ which pg_repack
/usr/bin/pg_repack
[postgres@testdb ~]$
```

However putting so much of efforts was not able to create extentions:
```
[postgres@testdb extension]$ psql -c "CREATE EXTENSION pg_repack" -d testdb
ERROR:  extension "pg_repack" is not available
DETAIL:  Could not open extension control file "/usr/pgsql-15/share/extension/pg_repack.control": No such file or directory.
HINT:  The extension must first be installed on the system where PostgreSQL is running.
[postgres@testdb extension]$
```

Then finally i decided to install it from binary package rather than source using [PostgreSQL repo](https://yum.postgresql.org/rpmchart/).
```
[root@testdb ~]# yum install https://download.postgresql.org/pub/repos/yum/15/redhat/rhel-8-x86_64/pg_repack_15-1.4.8-1.rhel8.x86_64.rpm
Last metadata expiration check: 0:02:44 ago on Thu 08 Feb 2024 12:27:36 PM +0545.
pg_repack_15-1.4.8-1.rhel8.x86_64.rpm                                                                                                                         65 kB/s | 107 kB     00:01
Dependencies resolved.
=============================================================================================================================================================================================
 Package                                       Architecture                            Version                                           Repository                                     Size
=============================================================================================================================================================================================
Installing:
 pg_repack_15                                  x86_64                                  1.4.8-1.rhel8                                     @commandline                                  107 k

Transaction Summary
=============================================================================================================================================================================================
Install  1 Package

Total size: 107 k
Installed size: 274 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                     1/1
  Installing       : pg_repack_15-1.4.8-1.rhel8.x86_64                                                                                                                                   1/1
  Running scriptlet: pg_repack_15-1.4.8-1.rhel8.x86_64                                                                                                                                   1/1
  Verifying        : pg_repack_15-1.4.8-1.rhel8.x86_64                                                                                                                                   1/1

Installed:
  pg_repack_15-1.4.8-1.rhel8.x86_64

Complete!
[root@testdb ~]#
```

Finally i was able to create the extention:
```
[postgres@testdb ~]$ psql -c "CREATE EXTENSION pg_repack" -d testdb
CREATE EXTENSION
[postgres@testdb ~]$
```

