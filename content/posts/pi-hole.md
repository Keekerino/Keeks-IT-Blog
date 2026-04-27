---
title: "Pi-hole Installation: Das DNS-Sinkhole für ein werbefreies Netz"
date: 2026-04-27T10:00:00+02:00
draft: false
author: "Keeks"
description: "So richtest du Pi-hole ein und bereitest deine Laufwerke für Backups vor."
featured_image: "/images/generic/Raspberry_Pi_Logo.svg"
tags: [RaspberryPi, Pi-hole, Security, Netzwerk, Adblocker]
categories: []
---



{{< notice "Installation von Pi-hole" >}}
In diesem Beitrag zeige ich dir die Installation von Pi-hole, die Einbindung einer leistungsstarken Blockliste und wie wir nebenbei noch den Speicher für Backup-Aufgaben vorbereiten. Somit können wir nun auch endlich Geräte, die keine Installation von Werbeblockern zulassen, effektiv von Werbung abschirmen, das Tracking der Websiten verhindern und durch die geringere Menge an abgerufenen Werbeseiten auch noch Daten sparen. Dies erhöht die Geschwindigkeit des Seitenaufbaus und hilft auch beim Kostenmanagement in  Datenspar-Tarifen.
{{< /notice >}}

<!--more-->

Nachdem die Grundeinrichtung unseres Raspberry Pis abgeschlossen ist, kommen wir nun zum eigentlichen Herzstück des Projekts: Wir verwandeln den kleinen Rechner in ein echtes DNS-Sinkhole. Das Ziel? Werbung und Tracker netzwerkweit blockieren, bevor sie überhaupt unsere Bildschirme erreichen.

## Pi-hole herunterladen und installieren

Die Installation von Pi-hole ist dank eines vorbereiteten Scripts denkbar einfach. Wir verbinden uns von einem im selben lokalen Netz stehenden Rechner per SSH-Befehl (ssh nutzername@ipadresse-des-pi) mit dem Pi und laden das Setup herunter:

```bash
curl -sSL https://install.pi-hole.net | bash
```

Daraufhin öffnet sich eine grafische Oberfläche (TUI-Text User Interface) direkt im Terminal. Hier können, bis auf eine Ausnahme, die meisten Standardeinstellungen einfach mit Enter bestätigt werden.

{{< image src="/images/raspberry-pi/PiDNS.png" alt="Pi-hole DNS Setup" caption="Auswahl der Upstream-DNS-Server im Setup des Pi" width="600px" float="center" >}}

Ein wichtiger Punkt ist jedoch die Wahl der Upstream DNS Provider. Anstatt die großen US-Datenkraken zu nutzen, trage ich hier die europäischen Server von [DNS4EU](https://joindns4.eu/for-public) für die Öffentlichkeit ein. Das schützt die Privatsphäre noch einmal deutlich besser. Alternative DNS-Provider findet ihr auch auf meiner [Links-Seite]({{< ref "links/" >}})  

## Dashboard und Router-Konfiguration
Nach Abschluss der Installation spuckt das Terminal das **Admin-Passwort** aus, welches unbedingt sicher notiert werden sollte. 
{{< expand "Hast du das Fenster verpasst oder zu schnell weggeklickt? " >}} 
Tipp: Mit dem Befehl 

    pihole -a -p 

im Terminal kannst du jederzeit ein neues Passwort vergeben! 
{{< /expand >}} 

Nun kannst du dich bequem über den Browser in die Weboberfläche einloggen, indem du die IP-Adresse oder den Hostnamen deines Pis aufrufst (z. B. http://**Deine IP-Adresse**/admin).

{{< image src="/images/raspberry-pi/PiConnect.png" alt="Pi-hole Web-Interface" caption="Pi-Hole Dashboard nach erfolgreicher Anmeldung" width="600px" float="center" >}}

Damit jetzt auch alle Geräte im Haushalt den Adblocker nutzen, müssen wir dem Router (z. B. der Fritz!Box) mitteilen, dass der Pi ab sofort die Autorität hat:

**Dazu in  die Netzwerkeinstellungen des Routers wechseln  und die IPv4- und IPv6-Adresse des Raspberry Pis als lokalen DNS-Server eintragen.**  

Diese beiden IP-Adressen haben wir bereits vorher konfiguriert. Sollte der Raspberry Pi noch keine feste IPv4 Adresse haben, so muss dies ebenfalls in den Einstellungen des lokalen Routers vorgenommen werden. Hier Beispielhaft im Interface der Fritz!Box: {{< image src="/images/raspberry-pi/sameIp.png" alt="Fritz!Box DNS-Einstellungen" caption="Feste IP-Konfiguration für Raspberry Pi (Beispielhaft an Fritz!Box)" width="600px" float="center" >}} 

Ab sofort laufen alle Anfragen durch den eigenen Filter.


## Die Filter schärfen: Hagezi Blockliste
Standardmäßig filtert Pi-hole schon gut was weg. Wir wollen es aber noch ein wenig verbessern. Das GitHub-Repository von Hagezi bietet hervorragende, aktuelle Blocklisten.

    Gehe im Pi-hole Dashboard auf Adlists.

    Füge folgende URL hinzu: https://cdn.jsdelivr.net/gh/hagezi/dns-blocklists@latest/adblock/pro.txt

    Klicke auf Speichern.

Damit die Liste aktiv wird, müssen wir die sogenannte "Gravity" (die Datenbank von Pi-hole) aktualisieren. Das geht entweder im Web-Interface

*Admin-Panel -> Tools -> Update Gravity*  

 oder bei bestehender SSH-Verbindung mit dem Befehl:

```Bash
sudo pihole -g
```

## Bonus: Hardware-Check und Backup-Speicher vorbereiten
Mein System läuft auf einem USB-Stick, während eine eingelegte Micro-SD-Karte als Backup-Ziel dienen soll. Zur Überwachung der [Smart-Werte ](https://de.wikipedia.org/wiki/Self-Monitoring,_Analysis_and_Reporting_Technology) meines Boot-mediums installiere ich noch die "Smartmontools".


```Bash
# Tools installieren
sudo apt update && sudo apt install smartmontools -y

# Blockgeräte anzeigen, um den Namen des Sticks (z.B. sda) zu finden
lsblk

# Gesundheitsstatus abfragen (Gerätebezeichnung anpassen!)
sudo smartctl -a -d scsi /dev/sda
```

### Micro-SD-Karte formatieren und einhängen
Nun machen wir die SD-Karte als Datenspeicher startklar.

```Bash
# 1. Karte aushängen (falls sie vom System automatisch gemountet wurde)
sudo umount /dev/mmcblk0p*

# 2. Für allgemeine Nutzung formatieren (ext4 Dateisystem, Label "DATEN")
sudo mkfs.ext4 -L "DATEN" /dev/mmcblk0p1

# 3. Ordnerstruktur für den Einhängepunkt (Mountpoint) erstellen
sudo mkdir -p /mnt/meine_sd

# 4. Micro-SD-Karte mounten
sudo mount /dev/mmcblk0p1 /mnt/meine_sd
```

Damit die Karte auch nach einem Neustart des Pis automatisch wieder zur Verfügung steht, verankern wir sie im System. Dafür brauchen wir zuerst die eindeutige ID (PARTUUID):

```Bash
sudo blkid /dev/mmcblk0p1
```

Kopiere dir die angezeigte PARTUUID (ohne die Anführungszeichen) und öffne die Dateisystemtabelle:

```Bash
sudo nano /etc/fstab
```

Füge am Ende der Datei folgende Zeile ein (ersetze **DEINE_ID** durch deine eigene ID):

    PARTUUID=DEINE_ID /mnt/meine_sd ext4 defaults,nofail 0 2

Speichere die Datei und teste die Konfiguration mit folgendem Befehl:

```Bash
sudo mount -a
```

Wenn hier keine Fehlermeldung erscheint: Herzlichen Glückwunsch! Dein System ist sauber konfiguriert, der Pi-hole läuft und der Speicher für kommende Projekte ist gesichert. In einem der nächsten Blog Posts konfiguriere ich dann noch unbound als rekursiven DNS-Resolver auf dem Raspberry Pi. Die perfekte Ergänzung zum Pi-Hole!


---