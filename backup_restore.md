
## ------------- Backup and Restore procedures for Databases ------------- ##

# All database backup commands should be run on the database server (server IrishIRL-2 for us).
# You can find current database servers by going to hosts file in the same directory.
# Use root user for that.

# First backup on the server should be full, not incremental. 
# For that reason, it is adviced to make it manually using (change IrishIRL and yolo.live for the correct data):

    sudo -u backup mysqldump agama > /home/backup/mysql/agama.sql
    sudo -u backup duplicity --no-encryption full /home//backup/mysql/ rsync://IrishIRL@backup.yolo.live//home/IrishIRL/mysql/


# Restore MySQL data from the backup (change IrishIRL and yolo.live for your correct data):
# NB! You should input the data to the machine with your working database! If you have master and slave setup, upload to the master machine.
# NB! You need to have sudo priveledges to do the current steps. As an example, I am doing those teps via my root account. Maybe there are more secure ways to do that.

    sudo -u backup duplicity --no-encryption restore rsync://IrishIRL@backup.yolo.live//home/IrishIRL/mysql/ /home/backup/restore/
    mysql agama < /home/backup/restore/agama.sql


## Checking MySQL changes
# In our current setup you will see the changes on your web page.
# However you can also go to mysql:
    
    mysql

# From there list the data you want to check, for example:

   mysql> SHOW DATABASES;
   mysql> USE agama;
   mysql> SHOW TABLES;
   mysql> SELECT * FROM item;



## ------------- Backup and Restore procedures for InfluxDB ------------- ##

# All database backup commands should be run on the InfluxDB server (server IrishIRL-1 for us).
# You can find current database servers by going to hosts file in the same directory.

# First backup on the server should be full, not incremental. 
# For that reason, it is adviced to make it manually using (change IrishIRL and yolo.live for the correct data):
   
    su backup
    influxd backup -portable -database telegraf /home/backup/influxdb
    duplicity --no-encryption full /home//backup/influxdb/ rsync://IrishIRL@backup.yolo.live//home/IrishIRL/influxdb/


# To restore the backup you will need to delete existing telegraf database first. 
# It also makes sense to stop the Telegraf service so that it doesn't recreate the database before you could restore it:
# PS. service telegraf start/ stop could require higher priveledges, so you may need to use those commands with sudo.

    su backup
    duplicity --no-encryption restore rsync://IrishIRL@backup.yolo.live//home/IrishIRL/influxdb/ /home/backup/restore

    service telegraf stop
    influx -execute 'DROP DATABASE telegraf'
    influxd restore -portable -database telegraf /home/backup/restore
    service telegraf start

## Checking Influx changes
# First you have to login into influx, then find the correct database, telegraf in our case. Show measurements and pick the correct measurement, for our case it is syslog.
# Last step is to select all from the syslog, but filter should have X mins depending on when your backup was made. We are using now() - ??m not to flood the terminal window.

    influx
    > show databases
      ..
      telegraf
      ..
    > use telegraf
    > show measurements
      ..
      syslog
      ..
    > select * from syslog where time > now() - ??m


