+++
title = 'Cisco Smart Satellite Manager (SSM) Server AlmaLinux Upgrade Overview'
date = 2024-07-22
draft = false
tags= ["CentOS", "AlmaLinux", "licensing", "UC", "SSM"]
+++

As most of the other Cisco Unified Communications products are requiring migration from the underlying CentOS to AlmaLinux, the on-premises satellite license server is no exception. Here are a few high level steps needed in order to migrate.

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Migrated Settings](#migrated-settings)
- [Prerequisites](#prerequisites)
  - [Upgrade Existing Version 8 Servers to 8-202404](#upgrade-existing-version-8-servers-to-8-202404)
- [Migration Steps](#migration-steps)
- [Post Migration Steps](#post-migration-steps)
- [References](#references)

## Overview
The Cisco SSM On-Prem Licenser Server is also running on CentOS, like other Cisco UC version 12x & 14x applications, and will have to be migrated as the underlying OS is nearing end of industry support.

**Migration Required.**  
**Direct upgrades _NOT_ supported.**

Current Version: **Older than 8-202404**  
Intermediate Version: **8-202404**  
Target Version: **9-202406**  


## Migrated Settings
*(Edited on 25 April 2025: The migration script and its SHA256 file are provided as part of the SSM OnPrem **9-202406** release ZIP file.)*  

* Network Configuration: This includes existing firewall zones and Custom routes of the interfaces if any. This will cover all the network configurations documented as part of SSM On-prem documentation, like ipv6 and dual NIC and so on.  
* TACACS+ Cli configuration (including the TACACS+ users and their roles).
* Docker network configurations (configured through On-prem console option).
* Message of the Day.
* NTP/Chrony Settings.
* Database: This includes the data of the application such as device details, secondary auth configuration, proxy, certificates used for product communication and custom browser certificates and so on. 

## Prerequisites
### Upgrade Existing Version 8 Servers to 8-202404
**Disregard HA steps if running a standalone server.**
1. Place upgrade files on both servers in patches folder `/var/files/patches`.
2. Disable HA on cluster.
> Note: Browser certificates are deleted when the HA teardown command is used.  
> * **onprem-console** 
> * **ha_teardown**
3. Issue the upgrade command on the first server and when complete, the second server.
**upgrade patches:SSM_On-Prem-8-202404_Upgrade.sh**
4. Reenable HA on the cluster.
> *Note: Browser certificates may need to be re-uplaoded.*

## Migration Steps
**Disregard HA steps if running a standalone server.**
1. Disable HA on the source CentOS cluster.
* **on-prem console**
* **ha_teardown**
2. Place migration script on primary source server in patches folder `/var/files/patches`. *(Included in **9-202406** release ZIP file.)*
   * **migrate_configs.sh  and migrate_configs.sh.sha256**
3. Run migration script.
   * **upgrade patches:migrate_configs.sh**
4.  Copy the backup files off the server from `/var/files/backups`.
   * Example file:  *onprem-migration-20240625214926.tar.gz*
5. Shut down source server(s) after new servers are staged.
6. Finish deployment on new (Alma Linux) server(s) with the exact same IP, Subnet, Gateway, and DNS as previous (CentOS) server.
7. Copy backup file to `/var/files/backups`.
8. Run the migration script on new Alma Linuxserver.
   * **on-prem console**
   * **upgrade patches:migrate_configs.sh**
9. Enter the name of the migration file when prompted.
10. After successful script execution, the server reboots.
11. Deploy secondary Alma Linux server.
* Enabling HA will copy over the primary server’s database to secondary.
12. Enable HA on cluster.
* Server 1: **onprem-console** -> **ha_generate keys**
* Server 2: **onprem-console** -> **ha_provision_standby**
* Server 1: **onprem-console** -> **ha_deploy**

## Post Migration Steps
It is recommended to perform SL sync (in admin console) and SLP sync (in licensing portal) after migration. This will make sure that all licensing information is up to date. 

## References
[Cisco Smart Software Manager On-Prem 8 Installation Guide Version 8 Release 202404](https://www.cisco.com/web/software/286326948/168202/SSM_On-Prem_8_Installation_Guide.pdf) 
[Cisco Smart Software Manager On-Prem Installation Guide Version 9 Release 202406](https://www.cisco.com/web/software/286326948/168546/SSM_On-Prem_9_Installation_Guide.pdf)  
[SSM On-Prem Migration Tool FAQ](https://www.cisco.com/web/software/286326948/168546/SSM_On-Prem_9_Migration_Tool_FAQ.pdf)  
[SSM On-Prem Migration Tool Quick Start Guide](https://www.cisco.com/web/software/286326948/168546/SSM_On-Prem_9_Migration_Tool_Quick_Start_Guide.pdf)
