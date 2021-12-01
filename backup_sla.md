Backup SLA:

1. Backup coverage: 
MySQL, InfluxDB, Ansible repo - should be backed up
From MySQL we backup sql dump, so data
From InfluxDB we backup data
From ansible repo we make a full mirror on the different server with full backup of all git history.

Grafana, DNS... - do not need to be backed up. 
Other services could be re-created running Ansible repo.

2. RPO: Creating local backups every night at 00:37 UTC and uploading them to backup server at 01:27 UTC, so acceptable data loss is around 24-25 hours.

3. Versioning and retention: backups are stored for 30 days, so 30 versions are stored.

4. Usability checks: Since I work in a quite big corporation, we have several people that are trying to restore everything from the backup each Monday and Thursday. In case something did not work out well, they report the error to the team that is answering for the particular service (in case of this task they report everything to me) and we are looking into the problem. 
All the results are also well documented and saved for the next 365 days as text documents using 3-2-1 rule.

5. Restoration criteria: in case of some data loss, in case our datacenters are down and we should build a backup server. Especially, in case of encryption due to malware (all the backup data should be also checked for the malwares though).

6. RTO: up to 2 hours

// Just a reminder for myself: in our real case there could be not more than 50MB of backups.

NB! It is a must to create two folders on a backup server before you start: mysql and influxdb. They will be used for the saving backups.
