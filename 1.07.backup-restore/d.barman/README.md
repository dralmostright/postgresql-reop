### Backup and Recovery Manager (BARMAN)

Barman (Backup and Recovery Manager) is an open-source administration tool for disaster recovery of PostgreSQL servers written in Python. It allows your organisation to perform remote backups of multiple servers in business critical environments to reduce risk and help DBAs during the recovery phase.

#### Configuring BARMAN

Requirement:
* One Server for BARMAN with internet access ( but for our testing case we will use the same server)
https://www.ashnik.com/postgresql-servers-back-up-disaster-recovery-with-barman/
https://cloud.google.com/alloydb/docs/omni/install-configure-barman
https://www.digitalocean.com/community/tutorials/how-to-back-up-restore-and-migrate-postgresql-databases-with-barman-on-centos-7
https://severalnines.com/blog/using-barman-backup-postgresql-overview/
* SSH-Key for passwordless authentication for Barman and Postgres users.
