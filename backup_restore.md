# Backup Restoration Policy

# Introduction

The purpose of this document is to explain the procedures that must be followed in case of a data loss.

**Attention: It is strictly prohibited to apply the steps listed below by an unauthorized person**

## Backup Locations and Procedures

This ansible repository is used to configure targeted virtual machines, that are listed in the `hosts` file. For starting the vm configuration process, `ansible-playbook infra.yaml` command can be executed.

The `mysql` and `influxdb` roles include tasks to initiate periodic backups, which can be found in `roles > mysql > templates > cron-mysql-backup` and `roles > influxdb > files > cron-influxdb-backup`. The periodic `mysql` task exports the content in `agama` database while `influxdb` task exports data in the `telegraf` database. The backup processes create two different backup types, `full` and `incremental`.

### MySQL and InfluxDB Dump 
* MySQL dump works on: `everyday at 20.00 UTC`
* MySQL dump destination path: `/home/backup/mysql/`
* InfluxDB dump works on: `everyday at 21.00 UTC`
* InfluxDB dump destination path: `/home/backup/influxdb`

The exported files are the main backup files and stored locally. These files are also used to create backups in remote destinations (backup server).

### Full Backups
* MySQL full backup schedule: `Every Sunday at 20.15 UTC`
* InfluxDB full backup schedule: `Every Sunday at 21.15 UTC`

### Incremental Backups
* MySQL incremental backup schedule: `Everyday excluding Sunday at 20.20 UTC`
* InfluxDB incremental backup schedule: `Everyday excluding  Sunday at 21.20 UTC`

### Local Backup Locations
* MySQL local backup file location : `home/backup/mysql/agama.sql`
* InfluxDB local backup files location: `/home/backup/influxdb/`

### Remote Backup Locations
* MySQL backup destination path on backup server: `/home/EmreKaratopuk/mysql`
* InfluxDB backup destination path on backup server: `/home/EmreKaratopuk/influxdb`

## Important Notes Before Following Recovery Procedures

* If the recovery files already exist in the `/home/backup/restore/*` destination directories, copying the backup data from the backup server will fail. In that case, there are several options:

    - Local backups can be renamed to avoid name conflicts. (eg. 'old' can be added in the end of the file names)

    - `--force` option can be used for duplicity to overwrite existing files.

    - Local backups can be deleted. **Please be careful** and do not delete the backups in the backup server.

* If the vms were configured around 15 minutes ago, an attempt of copying the backup data from the backup server may show an error indicating the backup server cannot identify the connecting machine thus cannot establish a connection. This is because the newly created public ssh keys in vms have not been copied to the backup server. This problem would be resolved once the lecture bot copies the public key to the `authorized_keys` file in the backup server. The expected problem resolution time is approximately 30 minutes.

## MySQL Recovery Steps

* Copy the backup file from the backup server to the ` /home/backup/restore/mysql` directory that is on the local device:

    ```bash
    sudo -u backup duplicity --no-encryption restore rsync://EmreKaratopuk@backup.istikbal.ek/mysql /home/backup/restore/mysql
    ```

* Copy the backup data to the mysql agama database with:
    ```bash
    sudo mysql agama < /home/backup/restore/mysql/agama.sql
    ```

In case backup server is not accessible, please check if `agama.sql` file exists in the `/home/backup/mysql/` directory as a local backup and try to restore the mysql data with the local backup file.

In the end of these steps, the mysql data should be recovered successfully. This can be confirmed by checking listed items on the agama website 

## InfluxDB Recovery Steps

* Copy the backup file from the backup server to the ` /home/backup/restore/influxdb` location in the second vm:

    ```bash
    sudo -u backup duplicity --no-encryption restore rsync://EmreKaratopuk@backup.istikbal.ek/influxdb /home/backup/restore/influxdb
    ```

* Stop the `telegraf service`:
    ```bash
    sudo service telegraf stop
    ```

* After stopping the telegraf service, you need to drop the `telegraf` database from influxdb. Delete `telegraf` database from `influxdb`
    ```bash
    influx -execute 'DROP DATABASE telegraf'
    ```
* Restore `influxdb` data:
    ```bash
    sudo influxd restore -portable -database telegraf /home/backup/restore/influxdb
    ```
* Start the `telegraf service`. This can be done by running this playbook as well. Command for manually starting:
    ```bash
    sudo service telegraf start
    ```

You may get these errors while restoring the database: 
```
error updating meta: DB metadata not changed. database may already exist
restore: DB metadata not changed. database may already exist
```
It's a known issue with InfluxDB restore, therefore errors can be ignored. You just need to make sure that the telegraf database is restored correctly.

In the end of these steps, the influxdb data should be recovered successfully. This can be confirmed by checking the `Syslog` dashboard on `Grafana`.

After following the required recovery steps above, please execute the `ansible-playbook infra.yaml` command to avoid bugs and preserve the consistency in vms. 