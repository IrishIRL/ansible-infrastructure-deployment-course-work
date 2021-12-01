
## ------------- Backup and Restore procedures for Databases ------------- ##

# All database backup commands should be run on the database server (server IrishIRL-2 for us).
# You can find current database servers by going to hosts file in the same directory.
# Use root user for that.

# First backup on the server should be full, not incremental. 
# For that reason, it is adviced to make it manually using (change IrishIRL and yolo.live for the correct data):

    sudo -u backup mysqldump agama > /home/backup/mysql/agama.sql
    sudo -u backup duplicity --no-encryption full /home//backup/mysql/ rsync://IrishIRL@backup.yolo.live//home/IrishIRL/mysql/


# Restore MySQL data from the backup (change IrishIRL and yolo.live for your correct data):

    sudo -u backup duplicity --no-encryption restore rsync://IrishIRL@backup.yolo.live//home/IrishIRL/mysql/ /home/backup/restore/
    mysql agama < /home/backup/restore/agama.sql


# In our current setup you will see the changes on your web page.
# However you can also go to mysql:
    
    mysql

# From there list the data you want to check, for example:

   mysql> SHOW DATABASES;
   mysql> SELECT * FROM agama;



## ------------- Backup and Restore procedures for InfluxDB ------------- ##

# All database backup commands should be run on the InfluxDB server (server IrishIRL-1 for us).
# You can find current database servers by going to hosts file in the same directory.

# First backup on the server should be full, not incremental. 
# For that reason, it is adviced to make it manually using (change IrishIRL and yolo.live for the correct data):
# PS. We are only backing up telegraf database in our systems.

    sudo influxd backup -portable -database telegraf /home/backup/influxdb
    sudo -u backup duplicity --no-encryption full /home//backup/influxdb/ rsync://IrishIRL@backup.yolo.live//home/IrishIRL/influxdb/


# To restore the backup you will need to delete existing telegraf database first. It also makes sense to stop the Telegraf service so that it doesn't recreate the database before you could restore it:
# Logically it should work, but I get some sort of strange error, will look at it more later...

    sudo -u backup duplicity --no-encryption restore rsync://IrishIRL@backup.yolo.live//home/IrishIRL/influxdb/ /home/backup/restore

    service telegraf stop
    influx -execute 'DROP DATABASE telegraf'
    influxd restore -portable -database telegraf /home/backup/restore
