### Backup and Recovery Manager (BARMAN)

Barman (Backup and Recovery Manager) is an open-source administration tool for disaster recovery of PostgreSQL servers written in Python. It allows your organisation to perform remote backups of multiple servers in business critical environments to reduce risk and help DBAs during the recovery phase.

#### Configuring BARMAN

Requirement:
* One Server for BARMAN 
* SSH-Key for passwordless authentication for Barman and Postgres users.

As we have Primary/Standby Server, we will use Standby server for BARMAN
<table>
<tr><td>S.No</td><td>Hostname</td><td>IP</td><td>Role</td></tr>
<tr><td>1</td><td>Node2</td><td>192.168.229.140</td><td>Barman Server (Backup Server)</td></tr>
<tr><td>2</td><td>Node1</td><td>192.168.229.138</td><td>PostgreSQL Database Server</td></tr>
</table>

Installing Barman:
```
[root@pgvm2 ~]# yum install https://download.postgresql.org/pub/repos/yum/common/redhat/rhel-8-x86_64/barman-3.9.0-1PGDG.rhel8.noarch.rpm
Last metadata expiration check: 0:21:18 ago on Thu 22 Feb 2024 11:29:34 AM +0545.
barman-3.9.0-1PGDG.rhel8.noarch.rpm                  39 kB/s |  54 kB     00:01
Dependencies resolved.
====================================================================================
 Package                 Arch       Version                 Repository         Size
====================================================================================
Installing:
 barman                  noarch     3.9.0-1PGDG.rhel8       @commandline       54 k
Installing dependencies:
 python3-argcomplete     noarch     1.9.3-6.el8             ol8_appstream      60 k
 python3-barman          noarch     3.9.0-1PGDG.rhel8       pgdg-common       515 k

Transaction Summary
====================================================================================
Install  3 Packages

Total size: 629 k
Total download size: 575 k
Installed size: 2.9 M
Is this ok [y/N]: y
Downloading Packages:
(1/2): python3-argcomplete-1.9.3-6.el8.noarch.rpm   374 kB/s |  60 kB     00:00
(2/2): python3-barman-3.9.0-1PGDG.rhel8.noarch.rpm  353 kB/s | 515 kB     00:01
------------------------------------------------------------------------------------
Total                                               393 kB/s | 575 kB     00:01
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                            1/1
  Installing       : python3-argcomplete-1.9.3-6.el8.noarch                     1/3
  Installing       : python3-barman-3.9.0-1PGDG.rhel8.noarch                    2/3
  Running scriptlet: barman-3.9.0-1PGDG.rhel8.noarch                            3/3
  Installing       : barman-3.9.0-1PGDG.rhel8.noarch                            3/3
  Running scriptlet: barman-3.9.0-1PGDG.rhel8.noarch                            3/3
/sbin/ldconfig: /etc/ld.so.conf.d/kernel-5.4.17-2102.201.3.el8uek.x86_64.conf:6: hwcap directive ignored

  Verifying        : python3-argcomplete-1.9.3-6.el8.noarch                     1/3
  Verifying        : python3-barman-3.9.0-1PGDG.rhel8.noarch                    2/3
  Verifying        : barman-3.9.0-1PGDG.rhel8.noarch                            3/3

Installed:
  barman-3.9.0-1PGDG.rhel8.noarch           python3-argcomplete-1.9.3-6.el8.noarch
  python3-barman-3.9.0-1PGDG.rhel8.noarch

Complete!
[root@pgvm2 ~]#
```
Now lets verify the version and change the password of the barman user.
```
[root@pgvm2 ~]# barman --version
3.9.0 Barman by EnterpriseDB (www.enterprisedb.com)
[root@pgvm2 ~]# passwd barman
Changing password for user barman.
New password:
BAD PASSWORD: The password fails the dictionary check - it is based on a dictionary word
Retype new password:
passwd: all authentication tokens updated successfully.
[root@pgvm2 ~]#
```
Next we will make passwordless ssh configuration:

```
[barman@pgvm2 ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/barman/.ssh/id_rsa):
Created directory '/var/lib/barman/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/barman/.ssh/id_rsa.
Your public key has been saved in /var/lib/barman/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:eptFWxHPa6Xxa+KENyeQ0icBss+ELM8zQdl6y2vzfKc barman@pgvm2.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|       oo.  .    |
|      o.+..  +   |
|     . =.. .. + .|
|      +.=.. o. * |
|       =S+o=..+ .|
|       .o+.o=.  .|
|      . . +. * + |
|       . B. +.*. |
|        + oo.Eo  |
+----[SHA256]-----+
[barman@pgvm2 ~]$
```
Next we will copy the public key and put it in autorized_keys file in pgvm1 server:
```
[barman@pgvm2 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub postgres@192.168.229.138
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/var/lib/barman/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
postgres@192.168.229.138's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'postgres@192.168.229.138'"
and check to make sure that only the key(s) you wanted were added.

[barman@pgvm2 ~]$ ssh postgres@192.168.229.138 date
Thu Feb 22 12:03:27 +0545 2024
[barman@pgvm2 ~]$
```

#### Configure Global Configuration File in Barman Server
Now lets make the changes on the configuration file as per our need:
```
[root@pgvm2 ~]# cp /etc/barman.conf /etc/barman.conf.22022024
[root@pgvm2 ~]# vi /etc/barman.conf
[root@pgvm2 ~]# cat /etc/barman.conf | grep -v ';' | grep '='
barman_user = barman
configuration_files_directory = /etc/barman.d
barman_home = /pgdata/pgbkp
log_file = /var/log/barman/barman.log
log_level = INFO
compression = gzip
basebackup_retry_times = 3
basebackup_retry_sleep = 30
minimum_redundancy = 3
retention_policy = RECOVERY WINDOW OF 4 WEEKS
[root@pgvm2 ~]#
```

Configure the server configuration file on the barman server:
```
[root@pgvm2 ~]# cd /etc/barman.d/
[root@pgvm2 barman.d]# vi pgdb-bkp.conf
[root@pgvm2 barman.d]# cat pgdb-bkp.conf
[pgdb-bkp]
description = "PostgreSQL Database Backup Config"
ssh_command = ssh postgres@192.168.229.138
conninfo = host=192.168.229.138 user=postgres port=5432 dbname=postgres password=welcome1
backup_options=concurrent_backup
backup_method=rsync
archiver=on
[root@pgvm2 barman.d]#
```

Now lets change the archive command:
```
[postgres@pgvm1 data]$ vi postgresql.conf
[postgres@pgvm1 data]$ psql
psql (14.8)
Type "help" for help.

postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show archive_command;
                                                      archive_command

----------------------------------------------------------------------------------------------------------------------------
 test ! -f postgres@192.168.229.140:/walarc/pg14/archive/%f && rsync -a %p postgres@192.168.229.140:/walarc/pg14/archive/%f
(1 row)

postgres=# 
```

Now lets verify the configuration:
```
[root@pgvm2 barman.d]# barman switch-wal --force pgdb-bkp
The WAL file 0000000300000000000000C9 has been closed on server 'pgdb-bkp'
[root@pgvm2 barman.d]#
[barman@pgvm2 ~]$ barman show-server pgdb-bkp
Server pgdb-bkp:
        active: True
        archive_command: test ! -f postgres@192.168.229.140:/walarc/pg14/archive/%f && rsync -a %p postgres@192.168.229.140:/walarc/pg14/archive/%f
        archive_mode: on
        archive_timeout: 0
        archived_count: 3
        archiver: True
        archiver_batch_size: 0
        autogenerate_manifest: False
        aws_profile: None
        aws_region: None
        azure_credential: None
        azure_resource_group: None
        azure_subscription_id: None
        backup_compression: None
        backup_compression_format: None
        backup_compression_level: None
        backup_compression_location: None
        backup_compression_workers: None
        backup_directory: /pgdata/pgbkp/pgdb-bkp
        backup_method: rsync
        backup_options: BackupOptions({'concurrent_backup'})
        bandwidth_limit: None
        barman_home: /pgdata/pgbkp
        barman_lock_directory: /pgdata/pgbkp
        basebackup_retry_sleep: 30
        basebackup_retry_times: 3
        basebackups_directory: /pgdata/pgbkp/pgdb-bkp/base
        check_timeout: 30
        checkpoint_timeout: 300
        compression: gzip
        config_file: /var/lib/pgsql/14/data/postgresql.conf
        conninfo: host=192.168.229.138 user=postgres port=5432 dbname=postgres password=*REDACTED*  create_slot: manual
        current_archived_wals_per_second: 0.02276656955105797083
        current_lsn: 0/CC000060
        current_size: 837225045
        current_xlog: 0000000300000000000000CC
        custom_compression_filter: None
        custom_compression_magic: None
        custom_decompression_filter: None
        data_checksums: off
        data_directory: /var/lib/pgsql/14/data
        description: PostgreSQL Database Backup Config
        disabled: False
        errors_directory: /pgdata/pgbkp/pgdb-bkp/errors
        failed_count: 0
        forward_config_path: False
        gcp_project: None
        gcp_zone: None
        has_backup_privileges: True
        hba_file: /var/lib/pgsql/14/data/pg_hba.conf
        hot_standby: on
        ident_file: /var/lib/pgsql/14/data/pg_ident.conf
        immediate_checkpoint: False
        included_files: ['/var/lib/pgsql/14/data/postgresql.auto.conf']
        incoming_wals_directory: /pgdata/pgbkp/pgdb-bkp/incoming
        is_archiving: True
        is_in_recovery: False
        is_superuser: True
        last_archived_time: 2024-02-22 02:04:14.237316-05:00
        last_archived_wal: 0000000300000000000000CB
        last_backup_maximum_age: None
        last_backup_minimum_size: None
        last_failed_time: None
        last_failed_wal: None
        last_wal_maximum_age: None
        max_incoming_wals_queue: None
        max_replication_slots: 10
        max_wal_senders: 10
        minimum_redundancy: 3
        msg_list: []
        name: pgdb-bkp
        network_compression: False
        parallel_jobs: 1
        parallel_jobs_start_batch_period: 1
        parallel_jobs_start_batch_size: 10
        passive_node: False
        path_prefix: None
        post_archive_retry_script: None
        post_archive_script: None
        post_backup_retry_script: None
        post_backup_script: None
        post_delete_retry_script: None
        post_delete_script: None
        post_recovery_retry_script: None
        post_recovery_script: None
        post_wal_delete_retry_script: None
        post_wal_delete_script: None
        postgres_systemid: 7134358889598855918
        pre_archive_retry_script: None
        pre_archive_script: None
        pre_backup_retry_script: None
        pre_backup_script: None
        pre_delete_retry_script: None
        pre_delete_script: None
        pre_recovery_retry_script: None
        pre_recovery_script: None
        pre_wal_delete_retry_script: None
        pre_wal_delete_script: None
        primary_checkpoint_timeout: 0
        primary_conninfo: None
        primary_ssh_command: None
        recovery_options: RecoveryOptions()
        recovery_staging_path: None
        replication_slot: None
        replication_slot_support: True
        retention_policy: RECOVERY WINDOW OF 4 WEEKS
        retention_policy_mode: auto
        reuse_backup: None
        server_txt_version: 14.8
        slot_name: None
        snapshot_disks: None
        snapshot_gcp_project: None
        snapshot_instance: None
        snapshot_provider: None
        snapshot_zone: None
        ssh_command: ssh postgres@192.168.229.138
        stats_reset: 2024-02-22 02:02:27.107966-05:00
        streaming_archiver: False
        streaming_archiver_batch_size: 0
        streaming_archiver_name: barman_receive_wal
        streaming_backup_name: barman_streaming_backup
        streaming_conninfo: host=192.168.229.138 user=postgres port=5432 dbname=postgres password=*REDACTED*        streaming_wals_directory: /pgdata/pgbkp/pgdb-bkp/streaming
        synchronous_standby_names: ['']
        tablespace_bandwidth_limit: None
        version_supported: True
        wal_compression: off
        wal_keep_size: 512MB
        wal_level: replica
        wal_retention_policy: MAIN
        wals_directory: /pgdata/pgbkp/pgdb-bkp/wals
        xlog_segment_size: 16777216
[barman@pgvm2 ~]$
```

Lets check the configuration:
```
[barman@pgvm2 ~]$ barman check pgdb-bkp
Server pgdb-bkp:
        WAL archive: FAILED (please make sure WAL shipping is setup)
        PostgreSQL: OK
        superuser or standard user with backup privileges: OK
        wal_level: OK
        directories: FAILED (/pgdata/pgbkp/pgdb-bkp: Permission denied)
        retention policy settings: OK
        backup maximum age: OK (no last_backup_maximum_age provided)
        backup minimum size: OK (0 B)
        wal maximum age: OK (no last_wal_maximum_age provided)
        wal size: OK (0 B)
        compression settings: OK
        failed backups: OK (there are 0 failed backups)
        minimum redundancy requirements: FAILED (have 0 backups, expected at least 3)
        ssh: OK (PostgreSQL server)
        systemid coherence: OK (no system Id stored on disk)
        archive_mode: OK
        archive_command: OK
        continuous archiving: OK
        archiver errors: OK
[barman@pgvm2 ~]$
```
As we can see some has failed hence lets make them correct:
```
[root@pgvm2 ~]# id barman
uid=974(barman) gid=971(barman) groups=971(barman)
[root@pgvm2 ~]# id postgres
uid=26(postgres) gid=26(postgres) groups=26(postgres)
[root@pgvm2 ~]# usermod barman -g barman -G postgres
[root@pgvm2 ~]# id barman
uid=974(barman) gid=971(barman) groups=971(barman),26(postgres)
[root@pgvm2 ~]#
[root@pgvm2 pgbkp]# usermod -g postgres -G barman postgres
[root@pgvm2 pgbkp]# id postgres
uid=26(postgres) gid=26(postgres) groups=26(postgres),971(barman)
[root@pgvm2 pgbkp]# id barman
uid=974(barman) gid=971(barman) groups=971(barman),26(postgres)
[root@pgvm2 pgbkp]#
[root@pgvm2 ~]# chmod -R 775 /pgdata/
[root@pgvm2 ~]#
```

To fix ```WAL archive: FAILED (please make sure WAL shipping is setup)``` we need to change the archive destination to barman config as below and some additional permission related issues:
```
[barman@pgvm2 ~]$ barman show-server pgdb-bkp | grep incoming
        incoming_wals_directory: /pgdata/pgbkp/pgdb-bkp/incoming
        max_incoming_wals_queue: None
[barman@pgvm2 ~]$

postgres=# select pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show archive_command;
                                                                 archive_command

--------------------------------------------------------------------------------------------------------------------------------------------------
 test ! -f barman@192.168.229.140:/pgdata/pgbkp/pgdb-bkp/incoming/%f && rsync -a %p barman@192.168.229.140:/pgdata/pgbkp/pgdb-bkp/incoming/%f
(1 row)

postgres=#
```

```
[root@pgvm2 barman.d]# chmod -R 777 /pgdata/
[root@pgvm2 barman.d]# sudo su - barman
[barman@pgvm2 ~]$ barman switch-wal --force --archive pgdb-bkp
The WAL file 0000000300000000000000D3 has been closed on server 'pgdb-bkp'
Waiting for the WAL file 0000000300000000000000D3 from server 'pgdb-bkp' (max: 30 seconds)
Processing xlog segments from file archival for pgdb-bkp
        0000000300000000000000CC
        0000000300000000000000CD
        0000000300000000000000CE
        0000000300000000000000CF
        0000000300000000000000D0
        0000000300000000000000D1
        0000000300000000000000D2
Processing xlog segments from file archival for pgdb-bkp
        0000000300000000000000D3
[barman@pgvm2 ~]$
[barman@pgvm2 ~]$ barman check pgdb-bkp
Server pgdb-bkp:
        PostgreSQL: OK
        superuser or standard user with backup privileges: OK
        wal_level: OK
        directories: OK
        retention policy settings: OK
        backup maximum age: OK (no last_backup_maximum_age provided)
        backup minimum size: OK (0 B)
        wal maximum age: OK (no last_wal_maximum_age provided)
        wal size: OK (0 B)
        compression settings: OK
        failed backups: OK (there are 0 failed backups)
        minimum redundancy requirements: FAILED (have 0 backups, expected at least 3)
        ssh: OK (PostgreSQL server)
        systemid coherence: OK (no system Id stored on disk)
        archive_mode: OK
        archive_command: OK
        continuous archiving: OK
        archiver errors: OK
[barman@pgvm2 ~]$
```

Now lets take the backup:
```
[barman@pgvm2 ~]$ barman list-server
pgdb-bkp - PostgreSQL Database Backup Config
[barman@pgvm2 ~]$ barman backup pgdb-bkp --wait
Starting backup using rsync-concurrent method for server pgdb-bkp in /pgdata/pgbkp/pgdb-bkp/base/20240222T133508
Backup start at LSN: 0/E2000028 (0000000300000000000000E2, 00000028)
This is the first backup for server pgdb-bkp
WAL segments preceding the current backup have been found:
        0000000300000000000000CC from server pgdb-bkp has been removed
        0000000300000000000000CD from server pgdb-bkp has been removed
        0000000300000000000000CE from server pgdb-bkp has been removed
        0000000300000000000000CF from server pgdb-bkp has been removed
        0000000300000000000000D0 from server pgdb-bkp has been removed
        0000000300000000000000D1 from server pgdb-bkp has been removed
        0000000300000000000000D2 from server pgdb-bkp has been removed
        0000000300000000000000D3 from server pgdb-bkp has been removed
        0000000300000000000000D4 from server pgdb-bkp has been removed
        0000000300000000000000D5 from server pgdb-bkp has been removed
        0000000300000000000000D6 from server pgdb-bkp has been removed
        0000000300000000000000D7 from server pgdb-bkp has been removed
        0000000300000000000000D8 from server pgdb-bkp has been removed
        0000000300000000000000D9 from server pgdb-bkp has been removed
        0000000300000000000000DA from server pgdb-bkp has been removed
        0000000300000000000000DB from server pgdb-bkp has been removed
        0000000300000000000000DC from server pgdb-bkp has been removed
        0000000300000000000000DD from server pgdb-bkp has been removed
        0000000300000000000000DE from server pgdb-bkp has been removed
        0000000300000000000000DF from server pgdb-bkp has been removed
        0000000300000000000000E0 from server pgdb-bkp has been removed
Starting backup copy via rsync/SSH for 20240222T133508
Copy done (time: 10 seconds)
This is the first backup for server pgdb-bkp
Asking PostgreSQL server to finalize the backup.
Backup size: 798.5 MiB
Backup end at LSN: 0/E2000138 (0000000300000000000000E2, 00000138)
Backup completed (start time: 2024-02-22 13:35:08.574429, elapsed time: 13 seconds)
Waiting for the WAL file 0000000300000000000000E2 from server 'pgdb-bkp'
Processing xlog segments from file archival for pgdb-bkp
        0000000300000000000000E1
        0000000300000000000000E2
        0000000300000000000000E2.00000028.backup
[barman@pgvm2 ~]$
```
Reporting Backups:
```
[barman@pgvm2 ~]$ barman list-backup pgdb-bkp
pgdb-bkp 20240222T133508 - Thu Feb 22 02:50:19 2024 - Size: 798.5 MiB - WAL Size: 0 B
[barman@pgvm2 ~]$ barman show-backup pgdb-bkp 20240222T133508
Backup 20240222T133508:
  Server Name            : pgdb-bkp
  System Id              : 7134358889598855918
  Status                 : DONE
  PostgreSQL Version     : 140008
  PGDATA directory       : /var/lib/pgsql/14/data

  Base backup information:
    Disk usage           : 798.5 MiB (798.5 MiB with WALs)
    Incremental size     : 798.5 MiB (-0.00%)
    Timeline             : 3
    Begin WAL            : 0000000300000000000000E2
    End WAL              : 0000000300000000000000E2
    WAL number           : 1
    WAL compression ratio: 99.90%
    Begin time           : 2024-02-22 02:50:08.490105-05:00
    End time             : 2024-02-22 02:50:19.990406-05:00
    Copy time            : 10 seconds
    Estimated throughput : 74.8 MiB/s
    Begin Offset         : 40
    End Offset           : 312
    Begin LSN            : 0/E2000028
    End LSN              : 0/E2000138

  WAL information:
    No of files          : 0
    Disk usage           : 0 B
    Last available       : 0000000300000000000000E2

  Catalog information:
    Retention Policy     : VALID
    Previous Backup      : - (this is the oldest base backup)
    Next Backup          : - (this is the latest base backup)
[barman@pgvm2 ~]$
```