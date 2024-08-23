+++
title = 'SSM On-Prem Upgrade Issue: "No Matching MAC Found"'
date = 2024-08-19
draft = false
tags= ["licensing", "UC", "SSM"]
+++

This document provides step-by-step instructions for upgrading Cisco's Smart Software Manager (SSM) On-Prem from version 7-202001 to 8-202102, addressing common issues like Message Authentication Code (MAC) mismatches and using SFTP for file transfers.

<!--more-->

## Contents
- [Contents](#contents)
- [Overview](#overview)
- [Caveats](#caveats)
- [Components Used](#components-used)
- [Overview](#overview-1)
- [Workaround](#workaround)
- [File Transfer for STIG](#file-transfer-for-stig)

## Overview
This document contains steps to upgrade the Smart Software Manager (SSM) On-Prem when the Installation Guide steps as shown in the image taken from the Smart Software Manager On-Prem 8-202006 Installation Guide are unsuccessful.

![installguiscreenshot](https://github.com/RMPRofit/blog/blob/gh-pages/post/images/installguide.png)

## Caveats
The SSM server must not be in Federal Information Processing Standard (FIPS) mode as there is no access to the Linux shell when this is enabled.

## Components Used
The information in this document is based on these software and hardware versions:
- SSM 7-202001
- SSM 8-202102
- Prime Collaboration Deployment 12.6.1.10000-21
- WinSCP 5.19.4 (Build 11829 2021-10-24)
- Putty 0.70

## Overview
There are instances when the SSM cannot connect to an SFTP server where an upgrade ISO is stored. This results in an error and the SSM is unable to be upgraded. The workaround steps will be performed using an SSM On-Prem server running version 7-202001 and upgrading to 8-202102.

In this scenario, we are using Cisco Prime Collaboration Deployment (PCD) as the SFTP server. Using the following commands, the SSM On-Prem fails to pull the upgrade ISO from the SFTP server.

```javascript
[admin@SSM-on-Prem~]$ onprem-console
SSM On-Prem Console 
>> copy admintsftp@192.168.0.98:/upgrade/SSM On-Prem 8-202102 upgrade.sh.sha256 patches:
Copying admintsftp@192.168.0.98:/upgrade/SSM_On-Prem_8-202102_upgrade.sh.sha256 to /var/files/patches
[sudo] password for admin:
Last login: Thu Oct 28 21:01:01 UTC 2021 on cron
FIPS mode initialized
Unable to negotiate with 192.168.0.98 port 22: no matching MAC found. Their offer: hmac-sha1

>> copy admintsftp@192.168.0.98:/upgrade/SSM On-Prem 8-202102 upgrade.sh patches:
Copying admintsftp@192.168.0.98:/upgrade/SSM_On-Prem_8-202102_upgrade.sh to /var/files/patches 
Last login: Thu Oct 28 21:05:53 UTC 2021 on pts/0 
FIPS mode initialized 
Unable to negotiate with 192.168.0.98 port 22: no matching MAC found. Their offer: hmac-sha1
```
## Workaround

You will now have to use an SFTP program like WinSCP to connect to SSM On-Prem and copy the upgrade files into the default Admin directory.

![OnPrem File Directory](https://github.com/RMProfit/blog/blob/main/content/post/images/onpremdirectory.png)

Verify the files are present after first exiting out of the On-Prem Console and back to the Linux prompt. Once verified, they will have to be moved to the patches directory.

```javascript
>> exit
[admin@SSM-On-Prem ~]$ pwd
/home/admin
[admin@SSM-On-Prem ~]$ ls
SSM_On-Prem_8-202102_upgrade.sh  SSM_On-Prem_8-202102_upgrade.sh.sha256
[admin@SSM-On-Prem ~]$ mv SSM_On-Prem_8-202102_upgrade.sh.sha256 /var/files/patches 
[admin@SSM-On-Prem ~]$ mv SSM_On-Prem_8-202102_upgrade.sh /var/files/patches
[admin@SSM-On-Prem ~]$ ls /var/files/patches
SSM_On-Prem_8-202102_upgrade.sh  SSM_On-Prem_8-202102_upgrade.sh.sha256
```
Now go back into the On-Prem Console and initiate the upgrade.

```javascript
[admin@SSM-On-Prem ~]$ onprem-console
SSM On-Prem Console
>> upgrade patches:SSM_On-Prem_8-202102_upgrade.sh
Upgrading system from patch file /var/files/patches/SSM_On-Prem_8-202102_upgrade.sh 
[sudo] password for admin:
Last login: Thu Oct 28 21:15:17 orc 2021
Verified OK
Creating directory /var/tmp/ssms_patch-8-202102 
Verifying archive integrity… 100%  All good.
Uncompressing SSM On-Prem 6.X.X to 8-202102 Upgrade Patch
```

After the upgrade, you should be able to log in to the GUI or the CLI to verify the upgrade was successful.

```javascript
[admin@SSM-On-Prem ~]$ onprem-console
SSM On-Prem Console
>> version
[sudo] password for admin:
Last login: Mon Nov 1 20:52:40 UTC 2021 on tty1
On-Prem version (current): 8-202102
On-Prem version (ISO): 7-202001
Patches installed: 8-202004 8-202006 8-202008 8-202010 8-202102
``` 
![guipostupgrade](https://github.com/RMProfit/blog/blob/main/content/post/images/guipostupgrade.png)

These steps have also been used to upgrade from 8-202102 to 8-202108.
```javascript
>> upgrade patches:SSM_On-Prem_8-202108_upgrade.sh
Upgrading system from patch file /var/files/patches/SSM_On-Prem_8-202108_upgrade.sh 
Last login: Mon Nov  1 18:32:35 UTC 2021 on pts/0
Verified OK
Creating directory /var/tmp/ssms_patch-8-202108
Verifying archive integrity...  100%   All good.
Uncompressing SSM On-Prem 6.X.X to 8-202108 Upgrade Patch  100%

[admin@SSM-On-Prem ~]$ onprem-console
SSM On-Prem Console
>> version
[sudo] password for admin:
Last login: Mon Nov  1 20:52:40 UTC 2021 on tty1
On-Prem version (current): 8-202108
On-Prem version (ISO): 7-202001
Patches installed: 8-202004 8-202006 8-202008 8-202010 8-202102 8-202012 8-202105 8-202108
```

## File Transfer for STIG

Use the following curl command to transfer the upgrade file using FTP or SCP protocol to On-Prem console that configured with STIG compliance.

```javascript
curl -k -u <username for server> <scp/ftp>://<Server's IP address>/<path>/<filename> -o /var/files/patches/<filename>
```
Example:
```javascript
>> curl -k -u ftp-user ftp://10.10.10.1/test-ftp.txt -o /var/files/patches/test-ftp.txt 
OR 
>> curl -k -u scpuser scp://10.10.10.1/localdisk/defaultRepo/banner.txt -o /var/files/patches/banner.txt
```
