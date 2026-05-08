---
title: "SAMBA Dateiserver Kurzanleitung"
date: 2026-04-12T12:24:59+02:00
draft: false
author: "Keeks"
description: "Wie man SAMBA einrichtet um im lokalen Netzwerk Dateien Freizugeben"
featured_image: "/images/generic/samba-server.svg"
tags: [Samba, Linux, RaspberryPi, Kurzanleitung]
categories: []
---



{{< notice "SAMBA Dateiserver" >}}
Diese Kurzanleitung dokumentiert die Einrichtung einer Dateifreigabe unter Samba. Dies dient als Basis für die Speicherung von Backups eines Raspberry Pi im lokalen Netzwerk.
{{< /notice >}}

<!--more-->

## Samba Server bereitstellen

1. Pakete installieren

```bash
# Installieren der benötigten Pakete
sudo apt update && sudo apt install samba -y

```

2. Verzeichnis erstellen und Berechtigungen setzen

```bash
# Verzeichnis erstellen
sudo mkdir -p /mnt/dein_pfad

# Besitzer auf den gewählten Nutzer setzen
sudo chown -R dein_nutzername:dein_nutzername /mnt/dein_pfad

# Schreibrechte für den Besitzer vergeben
sudo chmod -R 755 /mnt/dein_pfad

```

3. Serverkonfiguration anpassen

```bash
sudo nano /etc/samba/smb.conf

```

Folgende Parameter am Ende der Konfigurationsdatei einfügen und den Wert **path** an die eigene Verzeichnisstruktur anpassen:

```ini
[Pi-Media]
path = /mnt/dein_pfad
writeable = yes
browseable = yes
public = no

```

4. Passwort für Netzzugriff setzen und Dienst neu starten

Hinweis: Der hier verwendete Nutzername muss bereits als regulärer Systemnutzer auf dem System existieren.

```bash
# Samba-Passwort für den Nutzer vergeben
sudo smbpasswd -a dein_nutzername

# Samba-Dienst neu starten
sudo systemctl restart smbd

```

5. Netzwerkzugriff

Im Dolphin-Dateimanager oder einem alternativen Dateiexplorer kann nun der Pfad zur Freigabe eingegeben werden:

**Linux:** `smb://<name_deines_pi>/Pi-Media`

**Windows:** `\\<name_deines_pi>\Pi-Media`

{{< image src="/images/raspberry-pi/12.04.26-Backups/sambaFreigabe.png" alt="Samba Freigabe" caption="Beispiel Dolphin Explorer" width="400px" float="center" >}}

Die Freigabe kann anschließend zu den Lesezeichen (z. B. „Meine Orte“) hinzugefügt werden. So bleibt der Ordner dauerhaft im Dateimanager sichtbar und ist jederzeit verfügbar. Ab sofort steht der freigegebene Ordner im lokalen Netzwerk für Lese- und Schreibzugriffe auf anderen Geräten bereit.
