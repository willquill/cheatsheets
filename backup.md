# Backup Cheatsheet

## Synology

Follow [this](https://www.therapeuticshub.com/rsync-backup-synology/) guide to create the rsync user and SSH keys.

## Backup everything one time

`rsync -avr -e "ssh -p 22" /home/will rsync@10.1.20.91::NetBackup/lizard/once`

## Backup up with cron

Helpful cron link [here](https://arewasfasuoy.wordpress.com/2020/08/22/automated-backups-using-rsync-cron/).

Structure of cron:

```cron
* * * * * <command>
m      h      d      M      day_of_week <command>
(0-59) (0-23) (1-31) (1-12) (0-6) (Sunday=0) <command> 
```

`crontab -e`

```cron
# Backup to Synology
# Every day at 2 AM
0 2 * * * rsync -avr --delete --ignore-existing -e "ssh -p 22" /home/will rsync@10.1.20.91::NetBackup/lizard/daily
# Every Sunday at 3 AM
0 3 * * 0 rsync -avr --delete --ignore-existing -e "ssh -p 22" /home/will rsync@10.1.20.91::NetBackup/lizard/weekly
# Every 1st at 4 AM
0 4 1 * * rsync -avr --delete --ignore-existing -e "ssh -p 22" /home/will rsync@10.1.20.91::NetBackup/lizard/monthly
```
