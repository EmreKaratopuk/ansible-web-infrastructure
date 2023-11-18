# Backup Policy for Ansible Repository

## Introduction

This document serves as the backup policy for the Ansible repository, which includes various services such as nginx, uwsgi, bind9, agama, prometheus, pinger, grafana, mysql, and influxdb.

### Backup Coverage

The backup policy covers all services mentioned above, the amount of data backed up depends on the service type. There are two different services, configurable and recoverable. Configurable services include nginx, uwsgi, bind9, agama, prometheus, pinger, and grafana, and their configuration information will be backed up. The recoverable services include mysql and influxdb services, their data will be fully stored in multiple locations to ensure the integrity of these services is always preserved.

### Recovery Point Objective (RPO)

The RPO for this backup policy is defined as the maximum acceptable amount of data loss in the event of a disaster. In the context of this project, the RPO is set to encompass data generated within the past 12 hours.

### Versioning and Retention

Backups will be versioned to allow for easy identification and restoration of specific points in time. Each backup will be labeled with a unique identifier, such as a timestamp or version number. Backups will be created every Sunday and backups will be preserved for 6 months to ensure a comprehensive data restoration can be performed in case of a disaster.

### Usability Checks

Usability checks will be performed twice a month to ensure the integrity and restorability of the backups. These checks will involve verifying the backup files, testing the restoration process, and validating the recovered data. Any issues or discrepancies discovered during these checks will be promptly addressed to maintain the reliability of the backup system.


### Restoration Criteria

In the event of a disaster, the restoration process will prioritize the recreation of configurable services including nginx, uwsgi, bind9, agama, prometheus, pinger, grafana. These services will be recreated using the latest configuration files and any necessary dependencies. On the other hand, recoverable services, mysql and influxdb, will be restored using the backup files to ensure data consistency.

### Recovery Time Objective (RTO)

The overall goal is to minimize downtime and restore services as quickly as possible. The maximum RTO is set to 12 hours for this project.
