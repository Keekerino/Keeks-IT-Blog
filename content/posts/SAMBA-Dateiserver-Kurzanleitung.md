---
title: "SAMBA Dateiserver Kurzanleitung"
date: 2026-04-12T12:24:59+02:00
draft: false
author: "Keeks"
description: "Wie man SAMBA einrichtet um im lokalen Netzwerk Dateien Freizugeben"
featured_image: "/images/generic/Keeks4.png"
tags: [Samba, Linux, RaspberrPi, Kurzanleitung]
categories: []
---



{{< notice "SAMBA Dateiserver leicht gemacht" >}}
Hier mal quick and dirty wie man sich eine Dateifreigabe unter Samba einrichtet. Dies wird später noch sehr nützlich sein, um unsere Backups des Raspberrys  im loakeln Netz zu speichern.
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
2. Diese Konfiguration in die Konfigurationsdatei kopieren und **path** gegebenefalls anpassen 
```
    [Pi-Media]
    path = /mnt/dein_pfad
    writeable = yes
    browseable = yes
    public = no
```

## Passwort für Netzzugriff setzen

```bash

sudo smbpasswd -a <dein_nutzername>
sudo systemctl restart smbd
```

## Besitzer auf User setzen und Schreibrechte für den Besitzer vergeben



```bash
# sollte dein Verzeichnis nicht existieren, erstelle es
sudo mkdir /mnt/dein_pfad

# Besitzer setzen
sudo chown -R <dein_nutzername>:<dein_nutzername> /mnt/dein_pfad

# Schreibrechte für Besitzer vergeben
sudo chmod -R 755 /mnt/dein_pfad

```

Nun kann im Dolphin Dateimanager im Kontextmenü des Orderns 

**smb://<name_deines_pi>/Pi-Media** (Linux)

**\\<name_deines_pi>\Pi-Media** (Windows)

{{< image src="/images/raspberry-pi/12.04.26-Backups/sambaFreigabe.png" alt="Beispiel für App-Passwörter Menü" caption="" width="400px" float="center" >}}
die Datei zu "meinen Orten” hinzugefügt werden, womit der Ordner dauerhaft im Dateimanager angezeigt wird und verfügbar ist. 
Auch über alle anderen Geräte ist nun ein **Lesezugriff** auf die freigabe möglich.



---