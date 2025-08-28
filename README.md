# Infrastructure for Agama Web Application  

**Author:** Ali Emre Karatopuk  

## Table of Contents  

1. [Overview](#overview)  
2. [Infrastructure Goals](#infrastructure-goals)  
3. [System Architecture](#system-architecture)  
4. [Backups and Recovery](#backups-and-recovery)  


## Overview

This **Ansible** based infrastructure is designed to deploy, scale, and maintain the python based [Agama web application](https://github.com/hudolejev/agama) across multiple servers with **high availability (HA)**, **load balancing**, **database replication**, **local DNS resolution**, and **comprehensive monitoring and backup strategies**.  

The main objective is to ensure that end users can reliably access Agama while the system remains resilient, observable, and recoverable in case of failures.  

## Infrastructure Goals

The primary goals of this infrastructure are:

1. **Reliable Deployment of Agama Web Application**  
   - Automate the setup of all required services using Ansible roles.
   - Ensure consistent and repeatable deployments across multiple servers.

2. **High Availability (HA)**  
   - Use HAProxy and Keepalived to provide load balancing and failover between multiple Agama web servers.
   - Ensure minimal downtime in case of server failures.

3. **Database Replication and Reliability**  
   - MySQL is configured with replication to maintain up-to-date copies of the `agama` database across multiple nodes.
   - InfluxDB stores time-series metrics, with backup strategies to prevent data loss.

4. **Local DNS Resolution**  
   - Bind9 is used to provide local DNS services, including A and CNAME records for all components.
   - Enables internal name resolution without relying on external DNS.

5. **Monitoring and Observability**  
   - Prometheus and Grafana are used to monitor system and service metrics.
   - Exporters are installed for MySQL, InfluxDB, HAProxy, Bind9, Nginx, Keepalived, and Agama Docker containers.
   - Dashboards provide visibility into server health, application performance, and replication status.

6. **Automated Backups and Recovery**  
   - MySQL and InfluxDB backups are scheduled with both full and incremental strategies.
   - Backup files are stored locally and synced to a backup server.
   - Recovery procedures are documented [here](backup_restore.md) to ensure data can be restored in case of failure.

7. **Scalability**  
   - The infrastructure supports horizontal scaling of Agama web servers and related components.
   - New servers can be added by updating the inventory and rerunning the Ansible playbooks.

8. **Security and Access Control**  
   - Backup and database access is restricted to authorized users.
   - Sensitive credentials are handled securely using Ansible variables and templates.


## System Architecture

Below is an overview of the architecture and the relationships between the components:

### 1. Web Layer
- **Agama Web Application**  
  - Python-based web application deployed in Docker containers.
  - Can run multiple instances per host for scalability.
  - Serves dynamic content to end users.
- **Nginx**  
  - Acts as a reverse proxy for Agama containers.
  - Provides HTTPS support, caching, and static file delivery.

### 2. Load Balancing and High Availability Layer
- **HAProxy**  
  - Distributes incoming traffic across multiple Agama web servers.
  - Ensures even load distribution and improves fault tolerance.
- **Keepalived**  
  - Provides virtual IP failover between HAProxy nodes.
  - Ensures high availability in case one HAProxy instance fails.

### 3. DNS Layer
- **Bind9**  
  - Local DNS server providing internal name resolution for all services.
  - Hosts A, CNAME, and NS records for web servers, databases, and monitoring endpoints.

### 4. Data Layer
- **MySQL**  
  - Hosts the Agama database.
  - Configured with master-replica replication for high availability and reliability.
- **InfluxDB**  
  - Stores time-series metrics collected from the infrastructure.
  - Integrated with Telegraf for metrics collection from services and servers.

### 5. Monitoring Layer
- **Prometheus**  
  - Collects metrics from MySQL, InfluxDB, Bind9, HAProxy, Nginx, Keepalived, Agama containers and the host OS.
- **Grafana**  
  - Provides dashboards and visualizations for monitoring application health, system performance, and replication status.

### 6. Backup and Recovery Layer
- **Duplicity**  
  - Handles automated local backups for MySQL and InfluxDB.
  - Supports both full and incremental backups.
- **Backup Server**  
  - Main backup location to store copies of backup files for disaster recovery.

### 7. Network Layout
- Internal network uses private IPs managed via Bind9 for reliable internal communication.
- Services expose only necessary ports for monitoring, DNS, and database replication.
- Virtual IPs managed by Keepalived ensure effective failover for the load balancer.

## Backups and Recovery

The [SLA](backup_sla.md) provides an overview of backups and system uptime.

For detailed technical information on backup processes and system recovery, see the [backup restoration instructions](backup_restore.md).

