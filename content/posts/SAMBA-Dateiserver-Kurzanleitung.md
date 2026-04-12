---
title: "SAMBA Dateiserver Kurzanleitung"
date: 2026-04-12T12:24:59+02:00
draft: true
author: "Keeks"
description: "Wie man SAMBA einrichtet, um im lokalen Netzwerk Dateien Freizugeben"
featured_image: "/images/generic/Keeks4.png"
tags: [Samba, Linux, RaspberrPi, Kurzanleitung]
categories: []
---



{{< notice "SAMBA Dateiserver leicht gemacht" >}}
Hier quick and dirty wie man sich eine Dateifreigabe unter Samba einrichtet. Dies wird später noch sehr nützlich sein, um unsere backups im Netz zu speichern.
{{< /notice >}}

<!--more-->



## Samba Server bereitstellen

1. Installieren der Pakete


```bash

# installieren der benötigiten pakete
sudo apt update && sudo apt install samba -y

# Bearbeiten der Serverkonfiguration
sudo nano /etc/samba/smb.conf
```
2. Diese Konfiguration in die Konfigurationsdatei kopieren und **path** anpassen auf eigene Einstellungen
```
    [Pi-Media]
    path = /mnt/meine_sd/media
    writeable = yes
    browseable = yes
    public = no
```

## Passwort für Netzzugriff setzen

```bash

sudo smbpasswd -a s3bbl3r
sudo systemctl restart smbd
```

## Besitzer auf User setzen und Schreibrechte für den Besitzer vergeben

```bash
sudo chown -R s3bbl3r:s3bbl3r /mnt/meine_sd/media
sudo chmod -R 755 /mnt/meine_sd/media

```

Nun kann im Dolphin Dateimanager im Kontextmenü des Orderns 

**smb://<name_deines_pi>/Pi-Media** 

{{< image src="/images/raspberry-pi/12.04.26-Backups/sambaFreigabe.png" alt="Beispiel für App-Passwörter Menü" caption="" width="400px" float="center" >}}
die Datei zu "meinen Orten” hinzugefügt werden, womit der Ordner dauerhaft im Dateimanager angezeigt wird und verfügbar ist. 
Auch über alle anderen Geräte ist nun ein **Lesezugriff** auf die freigabe möglich.



---