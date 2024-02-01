## PostgreSQL
One of the most pobpular open source relational database system is PostgreSQL. And in this section we will be going through default and latest available version of PostgreSQL. On next sections we will also explore how we can install specific versions and others.

### What are we going to do in this section:
1. Installing PostgreSQL.
2. Securing PostgreSQL and Accessing the PostgreSQL Shell.
3. Installing the PostgreSQL Administration Package.

### Installing PostgreSQL

We will be installing the PostgreSQL in Oracle Linux 8 and the easiest way to install is using yum or dnf. On this step we need to sure that our machine can reach to internet. [PostgreSQL](https://www.postgresql.org/download/linux/redhat/) webside too has instructions.

```
[root@testdb etc]# dnf install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
[MIRROR] pgdg-redhat-repo-latest.noarch.rpm: Curl error (60): Peer certificate cannot be authenticated with given CA certificates for https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm [SSL certificate problem: unable to get local issuer certificate]
```

If you get this type of error, we can trun the ssl chek off in yum.conf file
```
[root@testdb etc]# vi /etc/yum.conf
[root@testdb etc]# cat /etc/yum.conf
[main]
gpgcheck=0
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
sslverify=false
[root@testdb etc]#
```
```
[root@testdb ~]# dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
pgdg-redhat-repo-latest.noarch.rpm               11 kB/s |  14 kB     00:01
Dependencies resolved.
================================================================================
 Package                Architecture Version           Repository          Size
================================================================================
Installing:
 pgdg-redhat-repo       noarch       42.0-38PGDG       @commandline        14 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 14 k
Installed size: 15 k
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : pgdg-redhat-repo-42.0-38PGDG.noarch                    1/1
  Verifying        : pgdg-redhat-repo-42.0-38PGDG.noarch                    1/1

Installed:
  pgdg-redhat-repo-42.0-38PGDG.noarch

Complete!
[root@testdb ~]#
```
Lets check the default available version in the AppStream repository using the following commands:
```
[root@testdb ~]# dnf module list postgresql
Last metadata expiration check: 0:00:49 ago on Thu 01 Feb 2024 01:11:15 PM +0545.
Oracle Linux 8 Application Stream (x86_64)
Name         Stream   Profiles             Summary
postgresql   9.6      client, server [d]   PostgreSQL server and client module
postgresql   10 [d]   client, server [d]   PostgreSQL server and client module
postgresql   12       client, server [d]   PostgreSQL server and client module
postgresql   13       client, server [d]   PostgreSQL server and client module
postgresql   15       client, server [d]   PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[root@testdb ~]#
```
As we can see postgresql 10 is default hence we will disable the default PostgreSQL repo using the following command:
```
[root@testdb ~]# dnf -qy module disable postgresql
[root@testdb ~]# dnf module list postgresql
Last metadata expiration check: 0:03:33 ago on Thu 01 Feb 2024 01:11:15 PM +0545.
Oracle Linux 8 Application Stream (x86_64)
Name         Stream     Profiles            Summary
postgresql   9.6 [x]    client, server [d]  PostgreSQL server and client module
postgresql   10 [d][x]  client, server [d]  PostgreSQL server and client module
postgresql   12 [x]     client, server [d]  PostgreSQL server and client module
postgresql   13 [x]     client, server [d]  PostgreSQL server and client module
postgresql   15 [x]     client, server [d]  PostgreSQL server and client module

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[root@testdb ~]#
```

Next, install the latest version of PostgreSQL by running the following command:
[root@testdb ~]# dnf install -y postgresql15-server
Last metadata expiration check: 0:04:24 ago on Thu 01 Feb 2024 01:11:15 PM +0545.
Dependencies resolved.
================================================================================
 Package               Arch     Version               Repository           Size
================================================================================
Installing:
 postgresql15-server   x86_64   15.5-2PGDG.rhel8      pgdg15              6.0 M
Installing dependencies:
 libicu                x86_64   60.3-2.el8_1          ol8_baseos_latest   8.8 M
 lz4                   x86_64   1.8.3-3.el8_4         ol8_baseos_latest   103 k
 postgresql15          x86_64   15.5-2PGDG.rhel8      pgdg15              1.6 M
 postgresql15-libs     x86_64   15.5-2PGDG.rhel8      pgdg15              294 k

Transaction Summary
================================================================================
Install  5 Packages

Total download size: 17 M
Installed size: 67 M
Downloading Packages:
(1/5): lz4-1.8.3-3.el8_4.x86_64.rpm              48 kB/s | 103 kB     00:02
(2/5): postgresql15-libs-15.5-2PGDG.rhel8.x86_6 113 kB/s | 294 kB     00:02
(3/5): postgresql15-15.5-2PGDG.rhel8.x86_64.rpm 344 kB/s | 1.6 MB     00:04
(4/5): libicu-60.3-2.el8_1.x86_64.rpm           846 kB/s | 8.8 MB     00:10
(5/5): postgresql15-server-15.5-2PGDG.rhel8.x86 860 kB/s | 6.0 MB     00:07
--------------------------------------------------------------------------------
Total                                           1.4 MB/s |  17 MB     00:11
PostgreSQL 15 for RHEL / Rocky / AlmaLinux 8 -  2.4 MB/s | 2.4 kB     00:00
Importing GPG key 0x08B40D20:
 Userid     : "PostgreSQL RPM Repository <pgsql-pkg-yum@lists.postgresql.org>"
 Fingerprint: D4BF 08AE 67A0 B4C7 A1DB CCD2 40BC A2B4 08B4 0D20
 From       : /etc/pki/rpm-gpg/PGDG-RPM-GPG-KEY-RHEL
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : postgresql15-libs-15.5-2PGDG.rhel8.x86_64              1/5
  Running scriptlet: postgresql15-libs-15.5-2PGDG.rhel8.x86_64              1/5
  Installing       : libicu-60.3-2.el8_1.x86_64                             2/5
  Running scriptlet: libicu-60.3-2.el8_1.x86_64                             2/5
  Installing       : lz4-1.8.3-3.el8_4.x86_64                               3/5
  Installing       : postgresql15-15.5-2PGDG.rhel8.x86_64                   4/5
  Running scriptlet: postgresql15-15.5-2PGDG.rhel8.x86_64                   4/5
  Running scriptlet: postgresql15-server-15.5-2PGDG.rhel8.x86_64            5/5
  Installing       : postgresql15-server-15.5-2PGDG.rhel8.x86_64            5/5
  Running scriptlet: postgresql15-server-15.5-2PGDG.rhel8.x86_64            5/5
  Verifying        : libicu-60.3-2.el8_1.x86_64                             1/5
  Verifying        : lz4-1.8.3-3.el8_4.x86_64                               2/5
  Verifying        : postgresql15-15.5-2PGDG.rhel8.x86_64                   3/5
  Verifying        : postgresql15-libs-15.5-2PGDG.rhel8.x86_64              4/5
  Verifying        : postgresql15-server-15.5-2PGDG.rhel8.x86_64            5/5

Installed:
  libicu-60.3-2.el8_1.x86_64
  lz4-1.8.3-3.el8_4.x86_64
  postgresql15-15.5-2PGDG.rhel8.x86_64
  postgresql15-libs-15.5-2PGDG.rhel8.x86_64
  postgresql15-server-15.5-2PGDG.rhel8.x86_64

Complete!
[root@testdb ~]#

We will install the postgresql-contrib component as well, which provides a set of useful extensions, additional utilities and features.
```
[root@testdb ~]# dnf install -y postgresql15-contrib postgresql15
Last metadata expiration check: 0:16:21 ago on Thu 01 Feb 2024 01:11:15 PM +0545.
Package postgresql15-15.5-2PGDG.rhel8.x86_64 is already installed.
Dependencies resolved.
================================================================================
 Package                Arch   Version                  Repository         Size
================================================================================
Installing:
 postgresql15-contrib   x86_64 15.5-2PGDG.rhel8         pgdg15            755 k
Installing dependencies:
 libxslt                x86_64 1.1.32-6.0.1.el8         ol8_baseos_latest 250 k
 perl-Carp              noarch 1.42-396.el8             ol8_baseos_latest  30 k
 perl-Data-Dumper       x86_64 2.167-399.el8            ol8_baseos_latest  58 k
 perl-Digest            noarch 1.17-395.el8             ol8_baseos_latest  27 k
 perl-Digest-MD5        x86_64 2.55-396.el8             ol8_baseos_latest  37 k
 perl-Encode            x86_64 4:2.97-3.el8             ol8_baseos_latest 1.5 M
 perl-Errno             x86_64 1.28-422.el8             ol8_baseos_latest  76 k
 perl-Exporter          noarch 5.72-396.el8             ol8_baseos_latest  34 k
 perl-File-Path         noarch 2.15-2.el8               ol8_baseos_latest  38 k
 perl-File-Temp         noarch 0.230.600-1.el8          ol8_baseos_latest  63 k
 perl-Getopt-Long       noarch 1:2.50-4.el8             ol8_baseos_latest  63 k
 perl-HTTP-Tiny         noarch 0.074-2.el8              ol8_baseos_latest  57 k
 perl-IO                x86_64 1.38-422.el8             ol8_baseos_latest 142 k
 perl-IO-Socket-IP      noarch 0.39-5.el8               ol8_baseos_latest  47 k
 perl-IO-Socket-SSL     noarch 2.066-4.module+el8.6.0+20623+f0897f98
                                                        ol8_appstream     298 k
 perl-MIME-Base64       x86_64 3.15-396.el8             ol8_baseos_latest  31 k
 perl-Mozilla-CA        noarch 20160104-7.0.1.module+el8.3.0+21136+b437fca9
                                                        ol8_appstream      15 k
 perl-Net-SSLeay        x86_64 1.88-2.module+el8.6.0+20623+f0897f98
                                                        ol8_appstream     379 k
 perl-PathTools         x86_64 3.74-1.el8               ol8_baseos_latest  90 k
 perl-Pod-Escapes       noarch 1:1.07-395.el8           ol8_baseos_latest  20 k
 perl-Pod-Perldoc       noarch 3.28-396.el8             ol8_baseos_latest  88 k
 perl-Pod-Simple        noarch 1:3.35-395.el8           ol8_baseos_latest 213 k
 perl-Pod-Usage         noarch 4:1.69-395.el8           ol8_baseos_latest  34 k
 perl-Scalar-List-Utils x86_64 3:1.49-2.el8             ol8_baseos_latest  68 k
 perl-Socket            x86_64 4:2.027-3.el8            ol8_baseos_latest  59 k
 perl-Storable          x86_64 1:3.11-3.el8             ol8_baseos_latest  98 k
 perl-Term-ANSIColor    noarch 4.06-396.el8             ol8_baseos_latest  46 k
 perl-Term-Cap          noarch 1.17-395.el8             ol8_baseos_latest  23 k
 perl-Text-ParseWords   noarch 3.30-395.el8             ol8_baseos_latest  18 k
 perl-Text-Tabs+Wrap    noarch 2013.0523-395.el8        ol8_baseos_latest  24 k
 perl-Time-Local        noarch 1:1.280-1.el8            ol8_baseos_latest  33 k
 perl-URI               noarch 1.73-3.el8               ol8_baseos_latest 116 k
 perl-Unicode-Normalize x86_64 1.25-396.el8             ol8_baseos_latest  82 k
 perl-constant          noarch 1.33-396.el8             ol8_baseos_latest  25 k
 perl-interpreter       x86_64 4:5.26.3-422.el8         ol8_baseos_latest 6.3 M
 perl-libnet            noarch 3.11-3.el8               ol8_baseos_latest 121 k
 perl-libs              x86_64 4:5.26.3-422.el8         ol8_baseos_latest 1.6 M
 perl-macros            x86_64 4:5.26.3-422.el8         ol8_baseos_latest  72 k
 perl-parent            noarch 1:0.237-1.el8            ol8_baseos_latest  20 k
 perl-podlators         noarch 4.11-1.el8               ol8_baseos_latest 118 k
 perl-threads           x86_64 1:2.21-2.el8             ol8_baseos_latest  61 k
 perl-threads-shared    x86_64 1.58-2.el8               ol8_baseos_latest  48 k
Enabling module streams:
 perl                          5.26
 perl-IO-Socket-SSL            2.066
 perl-libwww-perl              6.34

Transaction Summary
================================================================================
Install  43 Packages

Total download size: 13 M
Installed size: 39 M
Downloading Packages:
(1/43): perl-Carp-1.42-396.el8.noarch.rpm        17 kB/s |  30 kB     00:01
(2/43): perl-Data-Dumper-2.167-399.el8.x86_64.r  30 kB/s |  58 kB     00:01
(3/43): perl-Digest-1.17-395.el8.noarch.rpm      58 kB/s |  27 kB     00:00
(4/43): perl-Digest-MD5-2.55-396.el8.x86_64.rpm  61 kB/s |  37 kB     00:00
(5/43): libxslt-1.1.32-6.0.1.el8.x86_64.rpm      91 kB/s | 250 kB     00:02
(6/43): perl-Errno-1.28-422.el8.x86_64.rpm      150 kB/s |  76 kB     00:00
(7/43): perl-Exporter-5.72-396.el8.noarch.rpm    68 kB/s |  34 kB     00:00
(8/43): perl-File-Path-2.15-2.el8.noarch.rpm     73 kB/s |  38 kB     00:00
(9/43): perl-File-Temp-0.230.600-1.el8.noarch.r 102 kB/s |  63 kB     00:00
(10/43): perl-Getopt-Long-2.50-4.el8.noarch.rpm 127 kB/s |  63 kB     00:00
(11/43): perl-HTTP-Tiny-0.074-2.el8.noarch.rpm  102 kB/s |  57 kB     00:00
(12/43): perl-IO-1.38-422.el8.x86_64.rpm        180 kB/s | 142 kB     00:00
(13/43): perl-IO-Socket-IP-0.39-5.el8.noarch.rp  91 kB/s |  47 kB     00:00
(14/43): perl-MIME-Base64-3.15-396.el8.x86_64.r  69 kB/s |  31 kB     00:00
(15/43): perl-PathTools-3.74-1.el8.x86_64.rpm   169 kB/s |  90 kB     00:00
(16/43): perl-Encode-2.97-3.el8.x86_64.rpm      432 kB/s | 1.5 MB     00:03
(17/43): perl-Pod-Escapes-1.07-395.el8.noarch.r  45 kB/s |  20 kB     00:00
(18/43): perl-Pod-Perldoc-3.28-396.el8.noarch.r 192 kB/s |  88 kB     00:00
(19/43): perl-Pod-Usage-1.69-395.el8.noarch.rpm  84 kB/s |  34 kB     00:00
(20/43): perl-Socket-2.027-3.el8.x86_64.rpm     138 kB/s |  59 kB     00:00
(21/43): perl-Pod-Simple-3.35-395.el8.noarch.rp 239 kB/s | 213 kB     00:00
(22/43): perl-Scalar-List-Utils-1.49-2.el8.x86_  86 kB/s |  68 kB     00:00
(23/43): perl-Term-ANSIColor-4.06-396.el8.noarc 108 kB/s |  46 kB     00:00
(24/43): perl-Term-Cap-1.17-395.el8.noarch.rpm   56 kB/s |  23 kB     00:00
(25/43): perl-Storable-3.11-3.el8.x86_64.rpm    141 kB/s |  98 kB     00:00
(26/43): perl-Text-ParseWords-3.30-395.el8.noar  47 kB/s |  18 kB     00:00
(27/43): perl-Text-Tabs+Wrap-2013.0523-395.el8.  51 kB/s |  24 kB     00:00
(28/43): perl-Time-Local-1.280-1.el8.noarch.rpm  80 kB/s |  33 kB     00:00
(29/43): perl-URI-1.73-3.el8.noarch.rpm         244 kB/s | 116 kB     00:00
(30/43): perl-Unicode-Normalize-1.25-396.el8.x8 143 kB/s |  82 kB     00:00
(31/43): perl-constant-1.33-396.el8.noarch.rpm   53 kB/s |  25 kB     00:00
(32/43): perl-libnet-3.11-3.el8.noarch.rpm      163 kB/s | 121 kB     00:00
(33/43): perl-macros-5.26.3-422.el8.x86_64.rpm  152 kB/s |  72 kB     00:00
(34/43): perl-parent-0.237-1.el8.noarch.rpm      45 kB/s |  20 kB     00:00
(35/43): perl-libs-5.26.3-422.el8.x86_64.rpm    910 kB/s | 1.6 MB     00:01
(36/43): perl-podlators-4.11-1.el8.noarch.rpm   183 kB/s | 118 kB     00:00
(37/43): perl-threads-2.21-2.el8.x86_64.rpm     105 kB/s |  61 kB     00:00
(38/43): perl-threads-shared-1.58-2.el8.x86_64. 111 kB/s |  48 kB     00:00
(39/43): perl-Mozilla-CA-20160104-7.0.1.module+  33 kB/s |  15 kB     00:00
(40/43): perl-IO-Socket-SSL-2.066-4.module+el8. 290 kB/s | 298 kB     00:01
(41/43): perl-Net-SSLeay-1.88-2.module+el8.6.0+ 508 kB/s | 379 kB     00:00
(42/43): perl-interpreter-5.26.3-422.el8.x86_64 1.0 MB/s | 6.3 MB     00:06
(43/43): postgresql15-contrib-15.5-2PGDG.rhel8. 267 kB/s | 755 kB     00:02
--------------------------------------------------------------------------------
Total                                           930 kB/s |  13 MB     00:14
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Installing       : perl-Digest-1.17-395.el8.noarch                       1/43
  Installing       : perl-Digest-MD5-2.55-396.el8.x86_64                   2/43
  Installing       : perl-Data-Dumper-2.167-399.el8.x86_64                 3/43
  Installing       : perl-libnet-3.11-3.el8.noarch                         4/43
  Installing       : perl-URI-1.73-3.el8.noarch                            5/43
  Installing       : perl-Pod-Escapes-1:1.07-395.el8.noarch                6/43
  Installing       : perl-IO-Socket-IP-0.39-5.el8.noarch                   7/43
  Installing       : perl-Time-Local-1:1.280-1.el8.noarch                  8/43
  Installing       : perl-Mozilla-CA-20160104-7.0.1.module+el8.3.0+2113    9/43
  Installing       : perl-IO-Socket-SSL-2.066-4.module+el8.6.0+20623+f0   10/43
  Installing       : perl-Net-SSLeay-1.88-2.module+el8.6.0+20623+f0897f   11/43
  Installing       : perl-Term-ANSIColor-4.06-396.el8.noarch              12/43
  Installing       : perl-Term-Cap-1.17-395.el8.noarch                    13/43
  Installing       : perl-File-Temp-0.230.600-1.el8.noarch                14/43
  Installing       : perl-Pod-Simple-1:3.35-395.el8.noarch                15/43
  Installing       : perl-HTTP-Tiny-0.074-2.el8.noarch                    16/43
  Installing       : perl-podlators-4.11-1.el8.noarch                     17/43
  Installing       : perl-Pod-Perldoc-3.28-396.el8.noarch                 18/43
  Installing       : perl-Text-ParseWords-3.30-395.el8.noarch             19/43
  Installing       : perl-Pod-Usage-4:1.69-395.el8.noarch                 20/43
  Installing       : perl-MIME-Base64-3.15-396.el8.x86_64                 21/43
  Installing       : perl-Storable-1:3.11-3.el8.x86_64                    22/43
  Installing       : perl-Getopt-Long-1:2.50-4.el8.noarch                 23/43
  Installing       : perl-Errno-1.28-422.el8.x86_64                       24/43
  Installing       : perl-Socket-4:2.027-3.el8.x86_64                     25/43
  Installing       : perl-Encode-4:2.97-3.el8.x86_64                      26/43
  Installing       : perl-Exporter-5.72-396.el8.noarch                    27/43
  Installing       : perl-Scalar-List-Utils-3:1.49-2.el8.x86_64           28/43
  Installing       : perl-macros-4:5.26.3-422.el8.x86_64                  29/43
  Installing       : perl-parent-1:0.237-1.el8.noarch                     30/43
  Installing       : perl-Text-Tabs+Wrap-2013.0523-395.el8.noarch         31/43
  Installing       : perl-Unicode-Normalize-1.25-396.el8.x86_64           32/43
  Installing       : perl-File-Path-2.15-2.el8.noarch                     33/43
  Installing       : perl-IO-1.38-422.el8.x86_64                          34/43
  Installing       : perl-PathTools-3.74-1.el8.x86_64                     35/43
  Installing       : perl-constant-1.33-396.el8.noarch                    36/43
  Installing       : perl-threads-1:2.21-2.el8.x86_64                     37/43
  Installing       : perl-threads-shared-1.58-2.el8.x86_64                38/43
  Installing       : perl-libs-4:5.26.3-422.el8.x86_64                    39/43
  Installing       : perl-Carp-1.42-396.el8.noarch                        40/43
  Installing       : perl-interpreter-4:5.26.3-422.el8.x86_64             41/43
  Installing       : libxslt-1.1.32-6.0.1.el8.x86_64                      42/43
  Installing       : postgresql15-contrib-15.5-2PGDG.rhel8.x86_64         43/43
  Running scriptlet: postgresql15-contrib-15.5-2PGDG.rhel8.x86_64         43/43
  Verifying        : libxslt-1.1.32-6.0.1.el8.x86_64                       1/43
  Verifying        : perl-Carp-1.42-396.el8.noarch                         2/43
  Verifying        : perl-Data-Dumper-2.167-399.el8.x86_64                 3/43
  Verifying        : perl-Digest-1.17-395.el8.noarch                       4/43
  Verifying        : perl-Digest-MD5-2.55-396.el8.x86_64                   5/43
  Verifying        : perl-Encode-4:2.97-3.el8.x86_64                       6/43
  Verifying        : perl-Errno-1.28-422.el8.x86_64                        7/43
  Verifying        : perl-Exporter-5.72-396.el8.noarch                     8/43
  Verifying        : perl-File-Path-2.15-2.el8.noarch                      9/43
  Verifying        : perl-File-Temp-0.230.600-1.el8.noarch                10/43
  Verifying        : perl-Getopt-Long-1:2.50-4.el8.noarch                 11/43
  Verifying        : perl-HTTP-Tiny-0.074-2.el8.noarch                    12/43
  Verifying        : perl-IO-1.38-422.el8.x86_64                          13/43
  Verifying        : perl-IO-Socket-IP-0.39-5.el8.noarch                  14/43
  Verifying        : perl-MIME-Base64-3.15-396.el8.x86_64                 15/43
  Verifying        : perl-PathTools-3.74-1.el8.x86_64                     16/43
  Verifying        : perl-Pod-Escapes-1:1.07-395.el8.noarch               17/43
  Verifying        : perl-Pod-Perldoc-3.28-396.el8.noarch                 18/43
  Verifying        : perl-Pod-Simple-1:3.35-395.el8.noarch                19/43
  Verifying        : perl-Pod-Usage-4:1.69-395.el8.noarch                 20/43
  Verifying        : perl-Scalar-List-Utils-3:1.49-2.el8.x86_64           21/43
  Verifying        : perl-Socket-4:2.027-3.el8.x86_64                     22/43
  Verifying        : perl-Storable-1:3.11-3.el8.x86_64                    23/43
  Verifying        : perl-Term-ANSIColor-4.06-396.el8.noarch              24/43
  Verifying        : perl-Term-Cap-1.17-395.el8.noarch                    25/43
  Verifying        : perl-Text-ParseWords-3.30-395.el8.noarch             26/43
  Verifying        : perl-Text-Tabs+Wrap-2013.0523-395.el8.noarch         27/43
  Verifying        : perl-Time-Local-1:1.280-1.el8.noarch                 28/43
  Verifying        : perl-URI-1.73-3.el8.noarch                           29/43
  Verifying        : perl-Unicode-Normalize-1.25-396.el8.x86_64           30/43
  Verifying        : perl-constant-1.33-396.el8.noarch                    31/43
  Verifying        : perl-interpreter-4:5.26.3-422.el8.x86_64             32/43
  Verifying        : perl-libnet-3.11-3.el8.noarch                        33/43
  Verifying        : perl-libs-4:5.26.3-422.el8.x86_64                    34/43
  Verifying        : perl-macros-4:5.26.3-422.el8.x86_64                  35/43
  Verifying        : perl-parent-1:0.237-1.el8.noarch                     36/43
  Verifying        : perl-podlators-4.11-1.el8.noarch                     37/43
  Verifying        : perl-threads-1:2.21-2.el8.x86_64                     38/43
  Verifying        : perl-threads-shared-1.58-2.el8.x86_64                39/43
  Verifying        : perl-IO-Socket-SSL-2.066-4.module+el8.6.0+20623+f0   40/43
  Verifying        : perl-Mozilla-CA-20160104-7.0.1.module+el8.3.0+2113   41/43
  Verifying        : perl-Net-SSLeay-1.88-2.module+el8.6.0+20623+f0897f   42/43
  Verifying        : postgresql15-contrib-15.5-2PGDG.rhel8.x86_64         43/43

Installed:
  libxslt-1.1.32-6.0.1.el8.x86_64
  perl-Carp-1.42-396.el8.noarch
  perl-Data-Dumper-2.167-399.el8.x86_64
  perl-Digest-1.17-395.el8.noarch
  perl-Digest-MD5-2.55-396.el8.x86_64
  perl-Encode-4:2.97-3.el8.x86_64
  perl-Errno-1.28-422.el8.x86_64
  perl-Exporter-5.72-396.el8.noarch
  perl-File-Path-2.15-2.el8.noarch
  perl-File-Temp-0.230.600-1.el8.noarch
  perl-Getopt-Long-1:2.50-4.el8.noarch
  perl-HTTP-Tiny-0.074-2.el8.noarch
  perl-IO-1.38-422.el8.x86_64
  perl-IO-Socket-IP-0.39-5.el8.noarch
  perl-IO-Socket-SSL-2.066-4.module+el8.6.0+20623+f0897f98.noarch
  perl-MIME-Base64-3.15-396.el8.x86_64
  perl-Mozilla-CA-20160104-7.0.1.module+el8.3.0+21136+b437fca9.noarch
  perl-Net-SSLeay-1.88-2.module+el8.6.0+20623+f0897f98.x86_64
  perl-PathTools-3.74-1.el8.x86_64
  perl-Pod-Escapes-1:1.07-395.el8.noarch
  perl-Pod-Perldoc-3.28-396.el8.noarch
  perl-Pod-Simple-1:3.35-395.el8.noarch
  perl-Pod-Usage-4:1.69-395.el8.noarch
  perl-Scalar-List-Utils-3:1.49-2.el8.x86_64
  perl-Socket-4:2.027-3.el8.x86_64
  perl-Storable-1:3.11-3.el8.x86_64
  perl-Term-ANSIColor-4.06-396.el8.noarch
  perl-Term-Cap-1.17-395.el8.noarch
  perl-Text-ParseWords-3.30-395.el8.noarch
  perl-Text-Tabs+Wrap-2013.0523-395.el8.noarch
  perl-Time-Local-1:1.280-1.el8.noarch
  perl-URI-1.73-3.el8.noarch
  perl-Unicode-Normalize-1.25-396.el8.x86_64
  perl-constant-1.33-396.el8.noarch
  perl-interpreter-4:5.26.3-422.el8.x86_64
  perl-libnet-3.11-3.el8.noarch
  perl-libs-4:5.26.3-422.el8.x86_64
  perl-macros-4:5.26.3-422.el8.x86_64
  perl-parent-1:0.237-1.el8.noarch
  perl-podlators-4.11-1.el8.noarch
  perl-threads-1:2.21-2.el8.x86_64
  perl-threads-shared-1.58-2.el8.x86_64
  postgresql15-contrib-15.5-2PGDG.rhel8.x86_64

Complete!
[root@testdb ~]#
```

OEL does not automatically initialize or enable PostgreSQL hence we need to do it manually:
```
[root@testdb ~]# sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
Initializing database ... OK

[root@testdb ~]#
[root@testdb ~]# systemctl enable postgresql-15
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql-15.service → /usr/lib/systemd/system/postgresql-15.service.
[root@testdb ~]# systemctl start postgresql-15
[root@testdb ~]# systemctl status postgresql-15
● postgresql-15.service - PostgreSQL 15 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; enabled; vend>
   Active: active (running) since Thu 2024-02-01 13:32:19 +0545; 12s ago
     Docs: https://www.postgresql.org/docs/15/static/
  Process: 14632 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PG>
 Main PID: 14637 (postmaster)
    Tasks: 7 (limit: 12395)
   Memory: 17.5M
   CGroup: /system.slice/postgresql-15.service
           ├─14637 /usr/pgsql-15/bin/postmaster -D /var/lib/pgsql/15/data/
           ├─14639 postgres: logger
           ├─14640 postgres: checkpointer
           ├─14641 postgres: background writer
           ├─14643 postgres: walwriter
           ├─14644 postgres: autovacuum launcher
           └─14645 postgres: logical replication launcher

Feb 01 13:32:18 testdb.localdomain systemd[1]: Starting PostgreSQL 15 database >
Feb 01 13:32:18 testdb.localdomain postmaster[14637]: 2024-02-01 13:32:18.960 +>
Feb 01 13:32:18 testdb.localdomain postmaster[14637]: 2024-02-01 13:32:18.960 +>
Feb 01 13:32:19 testdb.localdomain systemd[1]: Started PostgreSQL 15 database s>
[root@testdb ~]#
```

By default, PostgreSQL listens on port 5432. We can check it with the following command:
```
[root@testdb ~]# ss -antpl | grep 5432
LISTEN 0      244        127.0.0.1:5432      0.0.0.0:*    users:(("postmaster",pid=14637,fd=7))
LISTEN 0      244            [::1]:5432         [::]:*    users:(("postmaster",pid=14637,fd=6))
[root@testdb ~]#
```

### Secure PostgreSQL and Access the PostgreSQL Shell

During the installation, PostgreSQL automatically creates a default user account named postgres and grants this user full superadmin privileges. Therefore, it is crucial to apply Linux and database passwords to the account.

```
[root@testdb ~]# passwd postgres
Changing password for user postgres.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@testdb ~]#
```
Switch over to this account with the su command.
```
[root@testdb ~]# sudo su - postgres
Last login: Thu Feb  1 13:48:48 +0545 2024 on pts/0
[postgres@testdb ~]$ which psql
/usr/bin/psql
[postgres@testdb ~]$ env | grep PG
PGDATA=/var/lib/pgsql/15/data
[postgres@testdb ~]$
```

Change the password that is required when the PostgreSQL user (postgres) connects over a network and we can use option -d if we want to change the password to specific database.
```
[postgres@testdb ~]$ psql -c "ALTER USER postgres WITH PASSWORD 'welcome1'"
ALTER ROLE
[postgres@testdb ~]$
```
The following command queries the PostgreSQL database for the current version.
```
[postgres@testdb ~]$ psql -c "SELECT version();"
                                                 version
---------------------------------------------------------------------------------------------------------
 PostgreSQL 15.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 8.5.0 20210514 (Red Hat 8.5.0-20), 64-bit
(1 row)

[postgres@testdb ~]$
```

Confirm you can access PostgreSQL by logging into the administrative postgres database.
```
[postgres@testdb ~]$ psql postgres
psql (15.5)
Type "help" for help.

postgres=# show hba_file;
              hba_file
------------------------------------
 /var/lib/pgsql/15/data/pg_hba.conf
(1 row)

postgres=#
```
