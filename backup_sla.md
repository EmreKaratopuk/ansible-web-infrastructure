# Backup Policy

## Introduction

This document serves as the backup policy for the Ansible repository, which includes various services such as nginx, bind9, agama, prometheus, pinger, keepalived, haproxy, grafana, mysql, and influxdb.

### Backup Coverage

The backup policy covers all services mentioned above, the amount of data backed up depends on the service type. There are two different services, configurable and recoverable. Configurable services include nginx, bind9, agama, prometheus, pinger, keepalived, haproxy and grafana, and their configuration information can be restored by running this ansible repository. The recoverable services include mysql and influxdb services, their data will be fully stored in multiple locations to ensure the integrity of these services is always preserved.

### Recovery Point Objective (RPO)

The RPO for this backup policy is defined as the maximum acceptable amount of data loss in the event of a disaster. In the context of this project, the RPO is set to data that generated within the past 24 hours.

### Versioning and Retention

Backups will be versioned to allow for easy identification and restoration of specific points in time. Each backup will be labeled with its creation timestamp and backup type. 

* MySQL full backup schedule: `Every Sunday at 20.15 UTC`
* InfluxDB full backup schedule: `Every Sunday at 21.15 UTC`
* MySQL incremental backup schedule: `Everyday excluding Sunday at 20.20 UTC`
* InfluxDB incremental backup schedule: `Everyday excluding Sunday at 21.20 UTC`

Backups will be preserved for 6 months to ensure a comprehensive data restoration can be performed in case of a disaster.

### Usability Checks

Usability checks will be performed `every 14 days` to ensure the integrity and restorability of the backups. These checks will involve verifying the backup files, testing the restoration process, and validating the recovered data. Any issues or discrepancies discovered during these checks will be promptly addressed to maintain the reliability of the backup system.


### Restoration Criteria

In the event of a disaster, the restoration process will prioritize the recreation of configurable services including nginx, uwsgi, bind9, agama, prometheus, pinger, grafana. These services will be recreated by running this Ansible repository. For recoverable services, please check the `backup_restore.md` file for the detailed procedure. 

### Recovery Time Objective (RTO)

The overall goal is to minimize downtime and restore services as quickly as possible. The maximum RTO is set to 2 hours for this project which involves recovering and testing data.
