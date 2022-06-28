---
title: Ranní meteogram
date: 2022-06-28
---

Pokud máte rádi meteogramy a systemd, pak jistě oceníte tento návod

<!--more-->

Český hydrometeorogický úřad vydává pravidelně několikrát za den meteogramy modelu ALADIN a tyto
poblikuje na [svém webu](https://www.chmi.cz/files/meteogramy.html).

Celý výstup je potom jeden jediný JPEG obrázek v predikovatelném formátu kde `XXXXXXXX` je datum,
`YY` je číslo vydání z daného dne a `ZZZ` je lokace pro kterou je meteogram vydán. Vizte formát a příklad:

> https://www.chmi.cz/files/portal/docs/meteo/ov/aladin/results/public/meteogramy/data/XXXXXXXXYY/ZZZ.png
> https://www.chmi.cz/files/portal/docs/meteo/ov/aladin/results/public/meteogramy/data/2022062800/355.png

## Licence meteogramu

Vezměte prosím na vědomí že meteogram je licencován. Aktuální licence říká že je nutné
vždy uvádět autora a není možné meteogram dále upravovat nebo sdílet! Vždy prosím
překontrolujte licenci na webu Českého hydrometerologického ústsavu.

## Script

Následující script stáhne první meteogram pro Prahu pro aktuální den a poté jej pošle na zadaný e-mail.

```bash
#!/usr/bin/env bash
DATE=$(date +"%Y%m%d00")
FILE="/tmp/meteogram${DATE}.png"
URL="https://www.chmi.cz/files/portal/docs/meteo/ov/aladin/results/public/meteogramy/data/${DATE}/355.png"
BODY="Meteogram Aladin pro Prahu ze dne $(date +"%d. %m. %Y"):\nZdroj: https://www.chmi.cz/files/meteogramy.html\n\n\n"

# Viz. https://www.chmi.cz/files/meteogramy.html
# Každá lokace má svoje číslo (například 355 pro Prahu)

curl -s $URL -o $FILE
echo -en $BODY | mail -r "quiiicq@gmail.com" -s "Ranní meteogram" -a $FILE -Ssendwait pdostal@pdostal.cz

# Odstraníme staré obrázky
find /tmp -daystart -maxdepth 1 -mtime +7 -type f -delete -iname "meteogram*.png"
```

## systemd timer

Místo `cron` můžete použít `systemd-timer`:

Na moderním systému funguje systemd i v uživatelském režimu.
Konfigurace se ukládá do `~/.config/systemd/user/` a příkazy
`systemctl` a `journalctl` se používají s parametrem `--user`.

```bash
$ mkdir -p ~/.config/systemd/user/
$ vim ~/.config/systemd/user/ranni-meteogram.service
[Unit]
Description=Downloads and send morning meteogram
Wants=ranni-meteogram.timer

[Service]
Type=oneshot
ExecStart=/home/pavel/bin/ranni-meteogram.sh

[Install]
WantedBy=multi-user.target


$ vim ~/.config/systemd/user/ranni-meteogram.timer
[Unit]
Description=Downloads and send morning meteogram
Requires=ranni-meteogram.service

[Timer]
Unit=ranni-meteogram.service
OnCalendar=*-*-* 06:30:00

[Install]
WantedBy=timers.target


$ systemctl --user daemon-reload
$ systemctl --user enable --now ranni-meteogram.timer
Created symlink /home/pavel/.config/systemd/user/timers.target.wants/ranni-meteogram.timer → /home/pavel/.config/systemd/user/ranni-meteogram.timer.
$ systemctl --user list-timers
NEXT                         LEFT     LAST                         PASSED       UNIT                         ACTIVATES
Wed 2022-06-29 06:30:00 CEST 17h left                                           ranni-meteogram.timer        ranni-meteogram.service
```

## Výsledek

Výsledkem je aktuální meteogram do Vaší schránky každé ráno.

