---
title: "Auslagerung von System-Logs in den Arbeitsspeicher mittels Log2Ram"
date: 2026-04-22T06:54:13Z
draft: false
author: "Keeks"
description: "Langlebigkeit von Speicherzellen garantieren"
featured_image: "/images/raspberry-pi/ram-memory.svg"
tags: [Linux, RaspberryPi, RAM]
categories: []
---



{{< notice "Lebensversicherung für die SD-Karte" >}}

Wer einen Raspberry Pi oder ein ähnliches System auf SD-Karten-Basis betreibt, sollte sich mit einem entscheidenden Nachteil dieser Konfiguration beschäftigen: Schreibzugriffe auf diesen Speicherbaustein sind der Tod auf Raten. Besonders die ständigen System-Protokolle (Log-Files) in /var/log fressen die Speicherzellen regelrecht auf... 
{{< /notice >}}

<!--more-->
Die elegante Lösung für dieses Problem heißt [Log2Ram](https://github.com/azlux/log2ram). Es schreibt die Logs schonend in den Arbeitsspeicher (RAM) und synchronisiert sie nur periodisch auf die Karte.

Nachdem wir unsere Backups bereits [sicher verstaut haben]({{< ref "posts/zeitgesteuerte-backups.md" >}}) , kümmern wir uns heute um den alltäglichen Verschleiß. Hier ist der saubere Weg, Log2Ram unter Debian 13 (Trixie) zu installieren, ohne sich das System zu zerschießen.
## Die Umgebung vorbereiten

Zuerst laden wir einmal die System-Variablen in unsere aktuelle Terminal-Sitzung. Das stellt sicher, dass das nachfolgende Skript ganz genau weiß, welche Debian-Version auf dem Pi läuft.
```Bash
# System-Infos in die aktuelle Session laden
. /etc/os-release
```
## Den Sicherheitsschlüssel (GPG) importieren

Da wir eine externe Quelle nutzen, müssen wir Debian mitteilen, dass wir dem Entwickler (Azlux) vertrauen. Dazu laden wir seinen offiziellen GPG-Schlüssel herunter und legen ihn sicher ab.
```Bash
wget -qO - https://azlux.fr/repo.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/azlux-archive-keyring.gpg
```

## Das Repository hinzufügen

Jetzt können wir das Repository sicher in unsere Paketverwaltung eintragen. Wir nutzen hier ein sogenanntes **"Here-Doc" (EOF)**, um die Datei sauber in einem Rutsch zu schreiben. Dieser Block kann im ganzen kopiert und ins terminal eingefügt werden:

```Bash
sudo tee /etc/apt/sources.list.d/azlux.list >/dev/null <<EOF
deb [signed-by=/usr/share/keyrings/azlux-archive-keyring.gpg] http://azlux.fr $VERSION_CODENAME main
EOF
```

## Der Debian 13 Trixie-Fix 

{{< expand "Warum brauchen wir diesen speziellen Schritt für Debian 13 (Trixie)?" >}} 
 Die Paketverwaltung stuft externe Quellen manchmal aus Sicherheitsgründen mit einer niedrigen Priorität ein, was dazu führt, dass die Installation blockiert wird. Mit einer sogenannten Prioritäts-Regel (Pin-Priority) verteilen wir einen VIP-Pass, der das System zwingt, für Log2Ram immer die neue Azlux-Quelle zu bevorzugen.  
{{< /expand >}}

Dieser Befehl prüft automatisch, ob Trixie genutzt wird, und setzt die Regel nur dann:

```Bash
[ "$VERSION_CODENAME" = trixie ] && sudo tee /etc/apt/preferences.d/log2ram.pref >/dev/null <<EOF
Package: log2ram
Pin: origin packages.azlux.fr
Pin-Priority: 1001
EOF
```

## Installation & Aktivierung

Jetzt bringen wir die Paketlisten auf den neuesten Stand, installieren Log2Ram und machen danach einen sauberen Neustart.

```Bash

# Listen aktualisieren
sudo apt update

# Log2Ram installieren
sudo apt install log2ram

# Neustart durchführen
sudo reboot
```
## Check & Feintuning

Nach dem Neustart loggen wir uns wieder ein und prüfen, ob die RAM-Disk erfolgreich eingerichtet wurde und ihren Dienst verrichtet:

```Bash
df -h | grep log2ram
```

Wenn hier ein Eintrag mit **/var/log** auftaucht, hat alles wunderbar funktioniert!

### Tipp zur Speichergröße:
Standardmäßig reserviert Log2Ram nur sparsame 40 MB im Arbeitsspeicher. Wenn dein Pi viel loggt (z. B. durch [Pi-Hole](https://pi-hole.net/), [Unbound](https://docs.pi-hole.net/guides/dns/unbound/) oder Webserver wie z.B. [Apache](https://httpd.apache.org/)), ist das schnell voll. Den Wert kann man ganz eifnach in der Konfiguration erhöhen. Daber sollte die Allokation **20 % des gesamten physischen RAMs nicht überschreiten**, um Out-of-Memory (OOM) Zustände des Betriebssystems zu verhindern.  

In dem hier genutzten Setup mit RaspberrryPi 4 mit 4GB RAM also **maximal 819 MB** *(20 % von 4.096 MB = ~819 MB)*. Auch sollte der Speicher nicht zu klein gewählt werden. Bei Überschreitung des konfigurierten SIZE-Limits wird das Verzeichnis schreibgeschützt. Dies führt zum sofortigen Stopp der Protokollierung und potenziellen Dienstausfällen. Mit **SIZE=256M** sind wir hier auf der sicheren Seite.  

Öffnen der Konfigurationsdatei im Terminal:

```Bash
sudo nano /etc/log2ram.conf
```

Dort nach der Zeile **SIZE=40M** suchen und passned zum System ändern (z. B. auf **SIZE=256M**).  
Danach die Datei speichern *(STRG + O im Editor Nano)* und einmal den Pi neustarten. 
```Bash
sudo reboot
```

Schon ist der Speicher auf dem RaspberryPi so eingerichtet, dass die ständigen Schreibzugriffe der Vergangenheit angehören. Die unzähligen Log-Einträge landen jetzt bequem im schnellen Arbeitsspeicher und werden nur noch ein- bis zweimal am Tag (oder beim Herunterfahren) gebündelt auf die SD-Karte geschrieben. Das verlängert die Lebensdauer des RasperryPi Speichers bedeutend.

---