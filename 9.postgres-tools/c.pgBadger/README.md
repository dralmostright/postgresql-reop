### pgBadger

pgBadger is a PostgreSQL performance analyzer, built for speed with fully detailed reports based on your PostgreSQL log files. pgBadger is a fast and easy tool to analyze your SQL traffic and create HTML5 reports with dynamics graphs. pgBadger is the perfect tool to understand the behavior of your PostgreSQL servers and identify which SQL queries need to be optimized.

We need to install it as it doesn't get shipped with PostgreSQL binaries. [pgBadger](https://yum.postgresql.org/packages/) is listed in postgresql common repository. Here we will be installing for rhel8 hence we will go for [RHEL8](https://download.postgresql.org/pub/repos/yum/common/redhat/rhel-8-x86_64/) repo.

Now lets install it:
```
[root@pgvm1 ~]# yum install https://download.postgresql.org/pub/repos/yum/common/redhat/rhel-8-x86_64/pgbadger-12.3-1PGDG.rhel8.noarch.rpm
PostgreSQL common RPMs for RHEL / Rocky 8 - x86_64  3.8 kB/s | 3.0 kB     00:00
PostgreSQL 15 for RHEL / Rocky 8 - x86_64           4.5 kB/s | 3.8 kB     00:00
PostgreSQL 14 for RHEL / Rocky 8 - x86_64           3.0 kB/s | 3.8 kB     00:01
pgbadger-12.3-1PGDG.rhel8.noarch.rpm                154 kB/s | 365 kB     00:02
Error:
 Problem: conflicting requests
  - nothing provides perl-Text-CSV_XS needed by pgbadger-12.3-1PGDG.rhel8.noarch from @commandline
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
[root@pgvm1 ~]# 
```
Oops!!

Seems it has some dependencies hence lets fix those:
```
[root@pgvm1 ~]# cd /etc/yum.repos.d/
[root@pgvm1 yum.repos.d]# curl -k -O https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/p/perl-Text-CSV_XS-1.40-1.el8.x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  145k  100  145k    0     0  50346      0  0:00:02  0:00:02 --:--:-- 50329
[root@pgvm1 yum.repos.d]# ls
docker-ce.repo         perl-Text-CSV_XS-1.40-1.el8.x86_64.rpm  pgdg-redhat-all.repo
oracle-linux-ol8.repo  pgadmin4.repo                           uek-ol8.repo
[root@pgvm1 yum.repos.d]# yum install perl-Text-CSV_XS-1.40-1.el8.x86_64.rpm
Last metadata expiration check: 0:02:47 ago on Thu 15 Feb 2024 07:02:17 PM +0545.
Error:
 Problem: conflicting requests
  - nothing provides perl(UNIVERSAL::isa) needed by perl-Text-CSV_XS-1.40-1.el8.x86_64 from @commandline
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
[root@pgvm1 yum.repos.d]# curl -k -O https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/Packages/p/perl-UNIVERSAL-isa-1.20171012-4.el8.noarch.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 24880  100 24880    0     0  16902      0  0:00:01  0:00:01 --:--:-- 16890
[root@pgvm1 yum.repos.d]# 
[root@pgvm1 yum.repos.d]# yum install perl-UNIVERSAL-isa-1.20171012-4.el8.noarch.rpm perl-Text-CSV_XS-1.40-1.el8.x86_64.rpm
Last metadata expiration check: 0:05:59 ago on Thu 15 Feb 2024 07:02:17 PM +0545.
Dependencies resolved.
====================================================================================
 Package                 Arch        Version                Repository         Size
====================================================================================
Installing:
 perl-Text-CSV_XS        x86_64      1.40-1.el8             @commandline      145 k
 perl-UNIVERSAL-isa      noarch      1.20171012-4.el8       @commandline       24 k

Transaction Summary
====================================================================================
Install  2 Packages

Total size: 170 k
Installed size: 332 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                            1/1
  Installing       : perl-UNIVERSAL-isa-1.20171012-4.el8.noarch                 1/2
  Installing       : perl-Text-CSV_XS-1.40-1.el8.x86_64                         2/2
  Running scriptlet: perl-Text-CSV_XS-1.40-1.el8.x86_64                         2/2
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

  Verifying        : perl-UNIVERSAL-isa-1.20171012-4.el8.noarch                 1/2
  Verifying        : perl-Text-CSV_XS-1.40-1.el8.x86_64                         2/2

Installed:
  perl-Text-CSV_XS-1.40-1.el8.x86_64   perl-UNIVERSAL-isa-1.20171012-4.el8.noarch

Complete!
[root@pgvm1 yum.repos.d]#
```

Finally we were able to install it:
```
[root@pgvm1 yum.repos.d]# yum install https://download.postgresql.org/pub/repos/yum/common/redhat/rhel-8-x86_64/pgbadger-12.3-1PGDG.rhel8.noarch.rpm
Last metadata expiration check: 0:08:43 ago on Thu 15 Feb 2024 07:02:17 PM +0545.
pgbadger-12.3-1PGDG.rhel8.noarch.rpm                227 kB/s | 365 kB     00:01
Dependencies resolved.
====================================================================================
 Package         Architecture  Version                    Repository           Size
====================================================================================
Installing:
 pgbadger        noarch        12.3-1PGDG.rhel8           @commandline        365 k

Transaction Summary
====================================================================================
Install  1 Package

Total size: 365 k
Installed size: 1.6 M
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                            1/1
  Installing       : pgbadger-12.3-1PGDG.rhel8.noarch                           1/1
  Running scriptlet: pgbadger-12.3-1PGDG.rhel8.noarch                           1/1
  Verifying        : pgbadger-12.3-1PGDG.rhel8.noarch                           1/1

Installed:
  pgbadger-12.3-1PGDG.rhel8.noarch

Complete!
[root@pgvm1 yum.repos.d]#
```
