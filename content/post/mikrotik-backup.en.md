---
title: Mikrotik backup script
date: 2024-05-27
tags:
  - mikrotik
  - network
---

I will show you how to daily backup your Mikrotik configuration. This uses `ssh`, `bash`, and `systemd-timer`.
The aim for this mini project is to be *as simple as possible* but it supports *multiple target devices*!

<!--more-->

## First of all, create the backup script (`~/bin/mikrotik-backup.sh`):

This script backs your Mikrotik up and then remove the backups older than 90 days.
You need to suply the device hostname and the directory where the backups will be saved.

```bash
#!/usr/bin/env bash
# Usage: ./mikrotik-backup <router-ip> <backup-dir>
ssh -4 $1 /export show-sensitive > $2/$1-$(date +"%Y%m%d%H%M%S").txt
find $2/ -name *.txt -type f -mtime +90 -exec rm '{}' \;
```

## Then create your systemd oneshot service (`~/.config/systemd/user/mikrotik-backup@.service`):

This unit counts with `~/mikrotik_backup/` destination directory.

```
[Unit]
Description=Back the Mikrotik up
Wants=mikrotik-backup@.timer

[Service]
Type=oneshot
ExecStart=%h/bin/mikrotik-backup.sh %i %h/mikrotik_backup/

[Install]
WantedBy=multi-user.target
```

## Lastly the systemd timer needs to be created (`~/.config/systemd/user/mikrotik-backup@.timer`):

The timer will be executed every day at 3:15.

```
[Unit]
Description=Back the Mikrotik up
Requires=mikrotik-backup@.service

[Timer]
Unit=mikrotik-backup@%i.service
OnCalendar=*-*-* 03:30:00

[Install]
WantedBy=timers.target
```

## SSH

This script needs to be able to `ssh` to the Mikrotik automagically, e.g. try `ssh gw.local /system/routerboard/print` and it should not require any other input.

If your username differs from the Mikrotik's username, just create `~/.ssh/config` file with:

```
Host *.local
  User MikrotikAdmin
```

## Enablement

1) To backup `gw.local` machine, just run:
  `systemctl --user enable --now mikrotik-backup@gw.local.timer`

2) You can inspect the timers by:
  `systemctl --user list-timers`

3) To see if the script ended successfully, run:
  `systemctl --user status mikrotik-backup@gw.local.service`

And that's it!

