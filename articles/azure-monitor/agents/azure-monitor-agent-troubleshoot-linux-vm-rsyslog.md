---
title: Syslog troubleshooting on AMA Linux Agent
description: Guidance for troubleshooting rsyslog issues on Linux virtual machines, scale sets with Azure Monitor agent and Data Collection Rules.
ms.topic: conceptual
ms.date: 5/3/2022
ms.custom: references_region
ms.reviewer: shseth
---
# Syslog issue troubleshooting guide for Azure Monitor Linux Agent
Here's how AMA collects syslog events:  

- AMA installs an output configuration for the system syslog daemon during the installation process. The configuration file specifies the way events flow between the syslog daemon and AMA.
- For `rsyslog` (most Linux distributions), the configuration file is `/etc/rsyslog.d/10-azuremonitoragent.conf`. For `syslog-ng`, the configuration file is `/etc/syslog-ng/conf.d/azuremonitoragent.conf`.
- AMA listens to a UNIX domain socket to receive events from `rsyslog` / `syslog-ng`. The socket path for this communication is `/run/azuremonitoragent/default_syslog.socket`
- The syslog daemon will use queues when AMA ingestion is delayed, or when AMA isn't reachable.
- AMA ingests syslog events via the aforementioned socket and filters them based on facility / severity combination from DCR configuration in `/etc/opt/microsoft/azuremonitoragent/config-cache/configchunks/`. Any `facility` / `severity` not present in the DCR will be dropped.
- AMA attempts to parse events in accordance with **RFC3164** and **RFC5424**. Additionally, it knows how to parse the message formats listed [here](./azure-monitor-agent-overview.md#data-sources-and-destinations).
- AMA identifies the destination endpoint for Syslog events from the DCR configuration and attempts to upload the events. 
	> [!NOTE]
	> AMA uses local persistency by default, all events received from `rsyslog` / `syslog-ng` are queued in `/var/opt/microsoft/azuremonitoragent/events` if they fail to be uploaded.
	
## Rsyslog data not uploaded due to full disk space issue on Azure Monitor Linux Agent

### Symptom
**Syslog data is not uploading**: When inspecting the error logs at `/var/opt/microsoft/azuremonitoragent/log/mdsd.err`, you'll see entries about *Error while inserting item to Local persistent store…No space left on device* similar to the following snippet:

```
2021-11-23T18:15:10.9712760Z: Error while inserting item to Local persistent store syslog.error: IO error: No space left on device: While appending to file: /var/opt/microsoft/azuremonitoragent/events/syslog.error/000555.log: No space left on device
```

### Cause
Linux AMA buffers events to `/var/opt/microsoft/azuremonitoragent/events` prior to ingestion. On a default Linux AMA install, this directory will take ~650MB of disk space at idle. The size on disk will increase when under sustained logging load. It will get cleaned up about every 60 seconds and will reduce back to ~650 MB when the load returns to idle.

### Confirming the issue of full disk
The `df` command shows almost no space available on `/dev/sda1`, as shown below:

```bash
   df -h
```
```output
Filesystem Size  Used Avail Use% Mounted on
udev        63G     0   63G   0% /dev
tmpfs       13G  720K   13G   1% /run
/dev/sda1   29G   29G  481M  99% /
tmpfs       63G     0   63G   0% /dev/shm
tmpfs      5.0M     0  5.0M   0% /run/lock
tmpfs       63G     0   63G   0% /sys/fs/cgroup
/dev/sda15 105M  4.4M  100M   5% /boot/efi
/dev/sdb1  251G   61M  239G   1% /mnt
tmpfs       13G     0   13G   0% /run/user/1000
```

The `du` command can be used to inspect the disk to determine which files are causing the disk to be full. For example:

```bash
   cd /var/log
   du -h syslog*
```
```output
6.7G    syslog
18G     syslog.1
```

In some cases, `du` may not report any significantly large files/directories. It may be possible that a [file marked as (deleted) is taking up the space](https://unix.stackexchange.com/questions/182077/best-way-to-free-disk-space-from-deleted-files-that-are-held-open). This issue can happen when some other process has attempted to delete a file, but there remains a process with the file still open. The `lsof` command can be used to check for such files. In the example below, we see that `/var/log/syslog` is marked as deleted, but is taking up 3.6 GB of disk space. It hasn't been deleted because a process with PID 1484 still has the file open.

```bash
   sudo lsof +L1
```   

```output
COMMAND   PID   USER   FD   TYPE DEVICE   SIZE/OFF NLINK  NODE NAME
none      849   root  txt    REG    0,1       8632     0 16764 / (deleted)
rsyslogd 1484 syslog   14w   REG    8,1 3601566564     0 35280 /var/log/syslog (deleted)
```

## Issue: rsyslog default configuration logs all facilities to /var/log/syslog
On some popular distros (for example Ubuntu 18.04 LTS), rsyslog ships with a default configuration file (`/etc/rsyslog.d/50-default.conf`) which will log events from nearly all facilities to disk at `/var/log/syslog`.

AMA doesn't rely on syslog events being logged to `/var/log/syslog`. Instead, it configures rsyslog to forward events over a socket directly to the azuremonitoragent service process (mdsd).

### Fix: Remove high-volume facilities from /etc/rsyslog.d/50-default.conf
If you're sending a high log volume through rsyslog, consider modifying the default rsyslog config to avoid logging these events to this location `/var/log/syslog`. The events for this facility would still be forwarded to AMA because of the config in `/etc/rsyslog.d/10-azuremonitoragent.conf`.

1. For example, to remove local4 events from being logged at `/var/log/syslog`, change this line in `/etc/rsyslog.d/50-default.conf` from this:
	```config
	*.*;auth,authpriv.none          -/var/log/syslog
	```

	To this (add local4.none;):

	```config
	*.*;local4.none;auth,authpriv.none          -/var/log/syslog
	```
2. `sudo systemctl restart rsyslog`

## Issue: Azure Monitor Linux Agent Event Buffer is Filling Disk
If you observe the `/var/opt/microsoft/azuremonitor/events` directory growing unbounded (10 GB or higher) and not reducing in size, [file a ticket](#file-a-ticket) with **Summary** as 'AMA Event Buffer is filling disk' and **Problem type** as 'I need help configuring data collection from a VM'.

[!INCLUDE [azure-monitor-agent-file-a-ticket](../../../includes/azure-monitor-agent/azure-monitor-agent-file-a-ticket.md)]
