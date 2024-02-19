### pgCluu (PostgreSQL Cluster utilization)
pgCluu is a Perl program used to perform a full audit of a PostgreSQL Cluster performances. It is divided in two parts, a collector used to grab statistics on the PostgreSQL cluster using psql and sar, a grapher that will generate all HTML and charts output.

All you need to run pgcluu is a modern perl distribution but this should be available if you are on a recent operating system. If you not only want to have statistics about a PostgreSQL instance but also want to have OS statistics you’ll need the sysstat package in addition (this should be available for your distribution). You can install pgcluu on the server where the PostgreSQL instance you want to monitor runs (as I will do for this post) or on a remote host. Installation is quite easy;

#### Some features of pgCluu
* Logging Monitoring: Monitors the logs of the PostgreSQL database and analyzes the data from these logs. This way, you can detect potential problems and errors.
* Total System Resources: Monitors and reports how your PostgreSQL cluster impacts system resources (CPU, memory, disk usage, etc.).
* Application Statistics: Shows application statistics of your PostgreSQL database and helps you optimize performance with these statistics.
* Database Performance Monitoring: Provides important information about database performance such as database transaction counts, connections, query statistics.
* Event Monitoring: Monitors the PostgreSQL database's activities (for example, long running queries) and provides information about these events.
* Customizable Reports: pgCluuoffers reports that can be customized according to users' needs.


First lets download the Source code.
```
[root@pgvm1 ~]# wget https://github.com/darold/pgcluu/archive/refs/tags/v3.4.tar.gz
--2024-02-19 15:48:26--  https://github.com/darold/pgcluu/archive/refs/tags/v3.4.tar.gz
Resolving github.com (github.com)... 20.205.243.166
Connecting to github.com (github.com)|20.205.243.166|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/darold/pgcluu/tar.gz/refs/tags/v3.4 [following]
--2024-02-19 15:48:26--  https://codeload.github.com/darold/pgcluu/tar.gz/refs/tags/v3.4
Resolving codeload.github.com (codeload.github.com)... 20.205.243.165
Connecting to codeload.github.com (codeload.github.com)|20.205.243.165|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/x-gzip]
Saving to: ‘v3.4.tar.gz’

v3.4.tar.gz             [    <=>             ]   1.09M  1.40MB/s    in 0.8s

2024-02-19 15:48:28 (1.40 MB/s) - ‘v3.4.tar.gz’ saved [1146357]

[root@pgvm1 ~]#
```

Untar it.
```
[root@pgvm1 ~]# tar -xvf v3.4.tar.gz
pgcluu-3.4/
pgcluu-3.4/.gitignore
pgcluu-3.4/ChangeLog
pgcluu-3.4/Dockerfile
pgcluu-3.4/LICENSE
pgcluu-3.4/MANIFEST
pgcluu-3.4/META.yml
pgcluu-3.4/Makefile.PL
....
......
pgcluu-3.4/resources/sorttable.js
pgcluu-3.4/resources/underscore.js
pgcluu-3.4/resources/w3.js
pgcluu-3.4/tools/
pgcluu-3.4/tools/README.updt_embedded_rsc
pgcluu-3.4/tools/updt_embedded_rsc.pl
[root@pgvm1 ~]#
```

Lastly lets intall it.
```
[root@pgvm1 pgcluu-3.4]# which perl
/usr/bin/perl
[root@pgvm1 pgcluu-3.4]# perl Makefile.PL
Can't locate ExtUtils/MakeMaker.pm in @INC (you may need to install the ExtUtils::MakeMaker module) (@INC contains: /usr/local/lib64/perl5 /usr/local/share/perl5 /usr/lib64/perl5/vendor_perl /usr/share/perl5/vendor_perl /usr/lib64/perl5 /usr/share/perl5) at Makefile.PL line 1.
BEGIN failed--compilation aborted at Makefile.PL line 1.
[root@pgvm1 pgcluu-3.4]# 
[root@pgvm1 pgcluu-3.4]# yum install perl-devel
Last metadata expiration check: 22:20:19 ago on Sun 18 Feb 2024 05:31:00 PM +0545.
Dependencies resolved.
================================================================================
 Package                     Arch   Version             Repository         Size
================================================================================
Installing:
 perl-devel                  x86_64 4:5.26.3-422.el8    ol8_appstream     600 k
Upgrading:
 perl-Errno                  x86_64 1.28-422.el8        ol8_baseos_latest  76 k
 perl-interpreter            x86_64 4:5.26.3-422.el8    ol8_baseos_latest 6.3 M
 perl-libs                   x86_64 4:5.26.3-422.el8    ol8_baseos_latest 1.6 M
Installing dependencies:
 cpp                         x86_64 8.5.0-20.0.2.el8    ol8_appstream      10 M
 dtrace                      x86_64 2.0.0-1.13.1.el8    ol8_UEKR6         4.5 M
 dwz                         x86_64 0.12-10.el8         ol8_appstream     109 k
 efi-srpm-macros             noarch 3-3.0.1.el8         ol8_appstream      22 k
 ghc-srpm-macros             noarch 1.4.2-7.el8         ol8_appstream     9.3 k
 go-srpm-macros              noarch 2-17.el8            ol8_appstream      13 k
 libpfm                      x86_64 4.13.0-4.el8        ol8_appstream     332 k
 ocaml-srpm-macros           noarch 5-4.el8             ol8_appstream     9.3 k
 openblas-srpm-macros        noarch 2-2.el8             ol8_appstream     7.9 k
 perl-CPAN-Meta-Requirements noarch 2.140-396.el8       ol8_appstream      37 k
 perl-CPAN-Meta-YAML         noarch 0.018-397.el8       ol8_appstream      34 k
 perl-ExtUtils-Command       noarch 1:7.34-1.el8        ol8_appstream      19 k
 perl-ExtUtils-Install       noarch 2.14-4.el8          ol8_appstream      46 k
 perl-ExtUtils-MakeMaker     noarch 1:7.34-1.el8        ol8_appstream     300 k
 perl-ExtUtils-Manifest      noarch 1.70-395.el8        ol8_appstream      36 k
 perl-ExtUtils-ParseXS       noarch 1:3.35-2.el8        ol8_appstream      83 k
 perl-JSON-PP                noarch 1:2.97.001-3.el8    ol8_appstream      68 k
 perl-Test-Harness           noarch 1:3.42-1.el8        ol8_appstream     279 k
 perl-srpm-macros            noarch 1-25.el8            ol8_appstream      11 k
 perl-version                x86_64 6:0.99.24-1.el8     ol8_appstream      67 k
 python-rpm-macros           noarch 3-45.el8            ol8_appstream      16 k
 python-srpm-macros          noarch 3-45.el8            ol8_appstream      16 k
 python3-rpm-macros          noarch 3-45.el8            ol8_appstream      15 k
 qt5-srpm-macros             noarch 5.15.3-1.el8        ol8_appstream      11 k
 redhat-rpm-config           noarch 125-1.0.1.el8       ol8_appstream      87 k
 rust-srpm-macros            noarch 5-2.el8             ol8_appstream     9.2 k
Installing weak dependencies:
 perl-CPAN-Meta              noarch 2.150010-396.el8    ol8_appstream     191 k
 perl-Encode-Locale          noarch 1.05-10.module+el8.3.0+7692+542c56f9
                                                        ol8_appstream      22 k
 perl-Time-HiRes             x86_64 4:1.9758-2.el8      ol8_appstream      61 k

Transaction Summary
================================================================================
Install  30 Packages
Upgrade   3 Packages

Total download size: 25 M
Is this ok [y/N]: y
Downloading Packages:
(1/33): efi-srpm-macros-3-3.0.1.el8.noarch.rpm   30 kB/s |  22 kB     00:00
(2/33): dwz-0.12-10.el8.x86_64.rpm              116 kB/s | 109 kB     00:00
(3/33): ghc-srpm-macros-1.4.2-7.el8.noarch.rpm   19 kB/s | 9.3 kB     00:00
(4/33): go-srpm-macros-2-17.el8.noarch.rpm       33 kB/s |  13 kB     00:00
(5/33): ocaml-srpm-macros-5-4.el8.noarch.rpm     41 kB/s | 9.3 kB     00:00
(6/33): libpfm-4.13.0-4.el8.x86_64.rpm          474 kB/s | 332 kB     00:00
(7/33): openblas-srpm-macros-2-2.el8.noarch.rpm  19 kB/s | 7.9 kB     00:00
(8/33): perl-CPAN-Meta-2.150010-396.el8.noarch. 433 kB/s | 191 kB     00:00
(9/33): cpp-8.5.0-20.0.2.el8.x86_64.rpm         4.4 MB/s |  10 MB     00:02
(10/33): perl-Encode-Locale-1.05-10.module+el8. 1.3 MB/s |  22 kB     00:00
(11/33): perl-CPAN-Meta-Requirements-2.140-396.  67 kB/s |  37 kB     00:00
(12/33): perl-CPAN-Meta-YAML-0.018-397.el8.noar 126 kB/s |  34 kB     00:00
(13/33): perl-ExtUtils-Command-7.34-1.el8.noarc  40 kB/s |  19 kB     00:00
(14/33): perl-ExtUtils-Install-2.14-4.el8.noarc  80 kB/s |  46 kB     00:00
(15/33): perl-ExtUtils-MakeMaker-7.34-1.el8.noa 628 kB/s | 300 kB     00:00
(16/33): perl-ExtUtils-ParseXS-3.35-2.el8.noarc 259 kB/s |  83 kB     00:00
(17/33): perl-ExtUtils-Manifest-1.70-395.el8.no  63 kB/s |  36 kB     00:00
(18/33): perl-Test-Harness-3.42-1.el8.noarch.rp 764 kB/s | 279 kB     00:00
(19/33): perl-JSON-PP-2.97.001-3.el8.noarch.rpm  93 kB/s |  68 kB     00:00
(20/33): perl-devel-5.26.3-422.el8.x86_64.rpm   1.8 MB/s | 600 kB     00:00
(21/33): perl-Time-HiRes-1.9758-2.el8.x86_64.rp  94 kB/s |  61 kB     00:00
(22/33): python-rpm-macros-3-45.el8.noarch.rpm  1.2 MB/s |  16 kB     00:00
(23/33): python-srpm-macros-3-45.el8.noarch.rpm 1.1 MB/s |  16 kB     00:00
(24/33): python3-rpm-macros-3-45.el8.noarch.rpm 1.0 MB/s |  15 kB     00:00
(25/33): perl-srpm-macros-1-25.el8.noarch.rpm    23 kB/s |  11 kB     00:00
(26/33): perl-version-0.99.24-1.el8.x86_64.rpm  239 kB/s |  67 kB     00:00
(27/33): redhat-rpm-config-125-1.0.1.el8.noarch 361 kB/s |  87 kB     00:00
(28/33): rust-srpm-macros-5-2.el8.noarch.rpm     47 kB/s | 9.2 kB     00:00
(29/33): perl-Errno-1.28-422.el8.x86_64.rpm     3.3 MB/s |  76 kB     00:00
(30/33): qt5-srpm-macros-5.15.3-1.el8.noarch.rp  23 kB/s |  11 kB     00:00
(31/33): perl-libs-5.26.3-422.el8.x86_64.rpm    2.4 MB/s | 1.6 MB     00:00
(32/33): dtrace-2.0.0-1.13.1.el8.x86_64.rpm     2.0 MB/s | 4.5 MB     00:02
(33/33): perl-interpreter-5.26.3-422.el8.x86_64 2.9 MB/s | 6.3 MB     00:02
--------------------------------------------------------------------------------
Total                                           3.7 MB/s |  25 MB     00:06
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Upgrading        : perl-libs-4:5.26.3-422.el8.x86_64                     1/36
  Upgrading        : perl-interpreter-4:5.26.3-422.el8.x86_64              2/36
  Installing       : perl-version-6:0.99.24-1.el8.x86_64                   3/36
  Installing       : python-srpm-macros-3-45.el8.noarch                    4/36
  Installing       : perl-CPAN-Meta-Requirements-2.140-396.el8.noarch      5/36
  Installing       : perl-ExtUtils-ParseXS-1:3.35-2.el8.noarch             6/36
  Installing       : perl-JSON-PP-1:2.97.001-3.el8.noarch                  7/36
  Installing       : perl-Time-HiRes-4:1.9758-2.el8.x86_64                 8/36
  Installing       : perl-Test-Harness-1:3.42-1.el8.noarch                 9/36
  Installing       : python-rpm-macros-3-45.el8.noarch                    10/36
  Installing       : python3-rpm-macros-3-45.el8.noarch                   11/36
  Installing       : perl-CPAN-Meta-YAML-0.018-397.el8.noarch             12/36
  Installing       : perl-CPAN-Meta-2.150010-396.el8.noarch               13/36
  Installing       : perl-Encode-Locale-1.05-10.module+el8.3.0+7692+542   14/36
  Installing       : perl-ExtUtils-Command-1:7.34-1.el8.noarch            15/36
  Installing       : perl-ExtUtils-Manifest-1.70-395.el8.noarch           16/36
  Installing       : rust-srpm-macros-5-2.el8.noarch                      17/36
  Installing       : qt5-srpm-macros-5.15.3-1.el8.noarch                  18/36
  Installing       : perl-srpm-macros-1-25.el8.noarch                     19/36
  Installing       : openblas-srpm-macros-2-2.el8.noarch                  20/36
  Installing       : ocaml-srpm-macros-5-4.el8.noarch                     21/36
  Installing       : libpfm-4.13.0-4.el8.x86_64                           22/36
  Running scriptlet: libpfm-4.13.0-4.el8.x86_64                           22/36
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

  Installing       : go-srpm-macros-2-17.el8.noarch                       23/36
  Installing       : ghc-srpm-macros-1.4.2-7.el8.noarch                   24/36
  Installing       : efi-srpm-macros-3-3.0.1.el8.noarch                   25/36
  Installing       : dwz-0.12-10.el8.x86_64                               26/36
  Installing       : redhat-rpm-config-125-1.0.1.el8.noarch               27/36
  Installing       : cpp-8.5.0-20.0.2.el8.x86_64                          28/36
  Running scriptlet: cpp-8.5.0-20.0.2.el8.x86_64                          28/36
  Installing       : dtrace-2.0.0-1.13.1.el8.x86_64                       29/36
  Running scriptlet: dtrace-2.0.0-1.13.1.el8.x86_64                       29/36
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored
Created symlink /etc/systemd/system/basic.target.wants/dtprobed.service → /usr/lib/systemd/system/dtprobed.service.
Created symlink /etc/systemd/system/basic.target.wants/dtrace-usdt.target → /usr/lib/systemd/system/dtrace-usdt.target.

  Installing       : perl-ExtUtils-Install-2.14-4.el8.noarch              30/36
  Installing       : perl-devel-4:5.26.3-422.el8.x86_64                   31/36
  Installing       : perl-ExtUtils-MakeMaker-1:7.34-1.el8.noarch          32/36
  Upgrading        : perl-Errno-1.28-422.el8.x86_64                       33/36
  Cleanup          : perl-Errno-1.28-419.el8.x86_64                       34/36
  Cleanup          : perl-interpreter-4:5.26.3-419.el8.x86_64             35/36
  Cleanup          : perl-libs-4:5.26.3-419.el8.x86_64                    36/36
  Running scriptlet: perl-libs-4:5.26.3-419.el8.x86_64                    36/36
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

  Verifying        : cpp-8.5.0-20.0.2.el8.x86_64                           1/36
  Verifying        : dwz-0.12-10.el8.x86_64                                2/36
  Verifying        : efi-srpm-macros-3-3.0.1.el8.noarch                    3/36
  Verifying        : ghc-srpm-macros-1.4.2-7.el8.noarch                    4/36
  Verifying        : go-srpm-macros-2-17.el8.noarch                        5/36
  Verifying        : libpfm-4.13.0-4.el8.x86_64                            6/36
  Verifying        : ocaml-srpm-macros-5-4.el8.noarch                      7/36
  Verifying        : openblas-srpm-macros-2-2.el8.noarch                   8/36
  Verifying        : perl-CPAN-Meta-2.150010-396.el8.noarch                9/36
  Verifying        : perl-CPAN-Meta-Requirements-2.140-396.el8.noarch     10/36
  Verifying        : perl-CPAN-Meta-YAML-0.018-397.el8.noarch             11/36
  Verifying        : perl-Encode-Locale-1.05-10.module+el8.3.0+7692+542   12/36
  Verifying        : perl-ExtUtils-Command-1:7.34-1.el8.noarch            13/36
  Verifying        : perl-ExtUtils-Install-2.14-4.el8.noarch              14/36
  Verifying        : perl-ExtUtils-MakeMaker-1:7.34-1.el8.noarch          15/36
  Verifying        : perl-ExtUtils-Manifest-1.70-395.el8.noarch           16/36
  Verifying        : perl-ExtUtils-ParseXS-1:3.35-2.el8.noarch            17/36
  Verifying        : perl-JSON-PP-1:2.97.001-3.el8.noarch                 18/36
  Verifying        : perl-Test-Harness-1:3.42-1.el8.noarch                19/36
  Verifying        : perl-Time-HiRes-4:1.9758-2.el8.x86_64                20/36
  Verifying        : perl-devel-4:5.26.3-422.el8.x86_64                   21/36
  Verifying        : perl-srpm-macros-1-25.el8.noarch                     22/36
  Verifying        : perl-version-6:0.99.24-1.el8.x86_64                  23/36
  Verifying        : python-rpm-macros-3-45.el8.noarch                    24/36
  Verifying        : python-srpm-macros-3-45.el8.noarch                   25/36
  Verifying        : python3-rpm-macros-3-45.el8.noarch                   26/36
  Verifying        : qt5-srpm-macros-5.15.3-1.el8.noarch                  27/36
  Verifying        : redhat-rpm-config-125-1.0.1.el8.noarch               28/36
  Verifying        : rust-srpm-macros-5-2.el8.noarch                      29/36
  Verifying        : dtrace-2.0.0-1.13.1.el8.x86_64                       30/36
  Verifying        : perl-Errno-1.28-422.el8.x86_64                       31/36
  Verifying        : perl-Errno-1.28-419.el8.x86_64                       32/36
  Verifying        : perl-interpreter-4:5.26.3-422.el8.x86_64             33/36
  Verifying        : perl-interpreter-4:5.26.3-419.el8.x86_64             34/36
  Verifying        : perl-libs-4:5.26.3-422.el8.x86_64                    35/36
  Verifying        : perl-libs-4:5.26.3-419.el8.x86_64                    36/36

Upgraded:
  perl-Errno-1.28-422.el8.x86_64      perl-interpreter-4:5.26.3-422.el8.x86_64
  perl-libs-4:5.26.3-422.el8.x86_64
Installed:
  cpp-8.5.0-20.0.2.el8.x86_64
  dtrace-2.0.0-1.13.1.el8.x86_64
  dwz-0.12-10.el8.x86_64
  efi-srpm-macros-3-3.0.1.el8.noarch
  ghc-srpm-macros-1.4.2-7.el8.noarch
  go-srpm-macros-2-17.el8.noarch
  libpfm-4.13.0-4.el8.x86_64
  ocaml-srpm-macros-5-4.el8.noarch
  openblas-srpm-macros-2-2.el8.noarch
  perl-CPAN-Meta-2.150010-396.el8.noarch
  perl-CPAN-Meta-Requirements-2.140-396.el8.noarch
  perl-CPAN-Meta-YAML-0.018-397.el8.noarch
  perl-Encode-Locale-1.05-10.module+el8.3.0+7692+542c56f9.noarch
  perl-ExtUtils-Command-1:7.34-1.el8.noarch
  perl-ExtUtils-Install-2.14-4.el8.noarch
  perl-ExtUtils-MakeMaker-1:7.34-1.el8.noarch
  perl-ExtUtils-Manifest-1.70-395.el8.noarch
  perl-ExtUtils-ParseXS-1:3.35-2.el8.noarch
  perl-JSON-PP-1:2.97.001-3.el8.noarch
  perl-Test-Harness-1:3.42-1.el8.noarch
  perl-Time-HiRes-4:1.9758-2.el8.x86_64
  perl-devel-4:5.26.3-422.el8.x86_64
  perl-srpm-macros-1-25.el8.noarch
  perl-version-6:0.99.24-1.el8.x86_64
  python-rpm-macros-3-45.el8.noarch
  python-srpm-macros-3-45.el8.noarch
  python3-rpm-macros-3-45.el8.noarch
  qt5-srpm-macros-5.15.3-1.el8.noarch
  redhat-rpm-config-125-1.0.1.el8.noarch
  rust-srpm-macros-5-2.el8.noarch

Complete!
[root@pgvm1 pgcluu-3.4]#
[root@pgvm1 pgcluu-3.4]#
```
Now lets make and install the binary
```
[root@pgvm1 pgcluu-3.4]# pwd
/root/pgcluu-3.4
[root@pgvm1 pgcluu-3.4]# perl Makefile.PL
Checking if your kit is complete...
Looks good
Generating a Unix-style Makefile
Writing Makefile for pgCluu
Writing MYMETA.yml and MYMETA.json
Done...

Now type 'make && make install'

[root@pgvm1 pgcluu-3.4]#
[root@pgvm1 pgcluu-3.4]# make && make install
bash: make: command not found...
Install package 'make' to provide command 'make'? [N/y] y


 * Waiting in queue...
 * Loading list of packages....
The following packages have to be installed:
 make-1:4.2.1-11.el8.x86_64     A GNU tool which simplifies the build process for users
Proceed with changes? [N/y] y


 * Waiting in queue...
 * Waiting for authentication...
 * Waiting in queue...
 * Downloading packages...
 * Requesting data...
 * Testing changes...
 * Installing packages...
cp pgcluu blib/script/pgcluu
"/usr/bin/perl" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/pgcluu
cp pgcluu_collectd blib/script/pgcluu_collectd
"/usr/bin/perl" -MExtUtils::MY -e 'MY->fixin(shift)' -- blib/script/pgcluu_collectd
Manifying 1 pod document

Manifying 1 pod document
Installing /usr/local/share/man/man1/pgcluu.1p
Installing /usr/local/bin/pgcluu
Installing /usr/local/bin/pgcluu_collectd
sh install_all.sh
install_all.sh: line 77: cd: /etc/apache2/conf-enabled/: No such file or directory
ln: failed to create symbolic link 'pgcluu.conf': File exists
Appending installation info to /usr/lib64/perl5/perllocal.pod
[root@pgvm1 pgcluu-3.4]#
[root@pgvm1 pgcluu-3.4]# which pgcluu_collectd
/usr/local/bin/pgcluu_collectd
[root@pgvm1 pgcluu-3.4]#
```

pgcluu is divided into two parts:
* The collector which is responsible for collecting the statistics: pgcluu_collectd
* The report generator which generates the reports out of the files the collector generated: pgcluu
To collect statistics start the pgcluu_collectd script as deamon:
```
[postgres@pgvm1 pgdata]$ pgcluu_collectd -D -i 10 /pgdata/pgcluu/
LOG: Setting retention from configuration file to 30
LOG: Detach from terminal with pid: 13185
[postgres@pgvm1 pgdata]$
```

This will collect statistics for the PostgreSQL instance you have the environment set for every 10 seconds and stores the results in the /pgdata/pgcluu/ directory:
```
[postgres@pgvm1 pgcluu]$ pwd
/pgdata/pgcluu
[postgres@pgvm1 pgcluu]$ ls -ltr | head
total 468
-rw-r--r-- 1 postgres postgres      0 Feb 19 16:07 pg_prepared_xact.csv
-rw-r--r-- 1 postgres postgres      0 Feb 19 16:07 pg_stat_replication.csv
-rw-r--r-- 1 postgres postgres      0 Feb 19 16:07 pg_stat_user_functions.csv
-rw-r--r-- 1 postgres postgres      0 Feb 19 16:07 pg_stat_xact_user_functions.csv
-rw-r--r-- 1 postgres postgres     61 Feb 19 16:07 pg_statio_user_sequences.csv
-rw-r--r-- 1 postgres postgres    425 Feb 19 16:07 pg_statio_user_indexes.csv
-rw-r--r-- 1 postgres postgres    449 Feb 19 16:07 pg_stat_user_indexes.csv
-rw-r--r-- 1 postgres postgres    461 Feb 19 16:07 pg_statio_user_tables.csv
-rw-r--r-- 1 postgres postgres    936 Feb 19 16:07 pg_stat_user_tables.csv
[postgres@pgvm1 pgcluu]$ 
```

After collecting logs for some time we can stop the log collection:
```
[postgres@pgvm1 pgcluu]$ pgcluu_collectd -k
LOG: Setting retention from configuration file to 30
OK: pgcluu_collectd exited with value 0
[postgres@pgvm1 pgcluu]$
```
Once we have some statistics collected we can generate a report:
```
[postgres@pgvm1 ~]$ mkdir -p /pgdata/reports/pgcluu
[postgres@pgvm1 ~]$ pgcluu -o /pgdata/reports/pgcluu/ /pgdata/pgcluu/
LOG: Setting retention from configuration file to 30
LOG: Setting output dir from configuration file to /var/lib/pgcluu/report
WARNING: No sar data file found. Consider using -S or --disable-sar command
line option or use -i / -I option to set the path to the data file.
Continuing normally without reporting sar statistics.
WARNING: No pidstat data file found.
Continuing normally without reporting pidstat statistics.
[postgres@pgvm1 ~]$
```
Once the reports are generated on the directory in html files.
Some snaps of the reports.
<img src='1.png' >
<img src='2.png' >
