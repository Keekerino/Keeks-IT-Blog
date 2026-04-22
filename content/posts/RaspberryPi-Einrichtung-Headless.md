---
title: "RaspberryPi OS-Lite (64-bit) Installation"
date: 2026-04-10T09:57:53Z
draft: false
author: "Keeks"
description: "Headless Installation über Netzwerk"
featured_image: "/images/generic/Raspberry_Pi_Logo.svg"
tags: [RaspberryPi, Terminal, DIY, Projekte]
categories: []
---



{{< notice "Die Reise beginnt" >}}
In diesem ersten Teil meiner neuen Raspberry-Pi-Serie zeige ich euch, wie ihr das System komplett „headless“ – also ohne Monitor und Tastatur – direkt über das Netzwerk aufsetzt, um somit einen möglichst geringen Ressourcenverbrauch des Gerätes sicherzustellen. Außerdem ist dies auch eine gute Übung für die Administration von Geräten über die SSH (Secure Shell).
Ein spannendes Thema auf dem Weg zu mehr Privatssphäre und Sicherheit im Netz.

<!--more-->



Das langfristige Ziel ist die Einrichtung eines [DNS-Sinkholes](https://en.wikipedia.org/wiki/DNS_sinkhole), dem [Pi-hole](https://pi-hole.net/), in Kombination mit einem lokalen DNS-Resolver [Unbound](https://docs.pi-hole.net/guides/dns/unbound/). Damit blockieren wir Werbung und Tracking netzwerkweit auf Systemebene. Dies schützt die Privatssphäre und beschleunigt durch lokales Caching die Ladezeiten ab der zweiten Anfrage deutlich. Heute aber erst einmal nur die Grundeinrichtung des Raspberry Pi. 

{{< image src="/images/raspberry-pi/pi4-board2.jpg" alt="Kontaktplättchen auf korrekten Chips platzieren" caption="" width="200px" float="right" >}}  

Nach dem aufkleben der korrekten Kühlplättchen auf die entsprechenden Chips und verschrauben des Kühlers kann es auch schon losgehen. 

Dazu hier ein kleines [Youtube Video von Geekworm](https://www.youtube.com/watch?v=tWb03sZk4lI).

{{< /notice >}}  



{{< expand "Benötigte Hardware" >}}



| Komponente | Details |
| :--- | :--- |
| **Board** | Raspberry Pi 4B |
| **Stromversorgung** | Original USB-C Netzteil |
| **Speicher** | SanDisk 32 GB Micro-SD-Karte und ein USB 3.0 Stick mit 64 GB |
| **Kühlung** | Passivkühler „Armor Case“ von BerryBase |



{{< /expand >}}

## Den Raspberry Pi Imager vorbereiten

Zuerst laden wir das offizielle Tool von der [Raspberry Pi Webseite](https://www.raspberrypi.com/software/) herunter. Da ich ein Arch-Linux-System (CachyOS) nutze, lade ich das AppImage herunter. Damit die Datei ausgeführt werden kann, müssen die entsprechenden  Berechtigungen im Terminal angepasst werden 

Terminal  öffnen (**STRG+ALT+T**):

```bash

# In das Verzeichnis mit der heruntergeladenen Datei wechseln
cd Downloads/

# Die Datei ausführbar machen
chmod +x imager_2.0.7_amd64.AppImage

# Das Programm mit Root-Rechten starten
sudo ./imager_2.0.7_amd64.AppImage

```

## Das Betriebssystem konfigurieren

Dazu muss nun das Medium, auf welches das Betriebssystem installiert werden soll im PC eingesteckt sein. In unserem Fall ist das der 64 GB USB 3.0 Stick von SanDisk.
In der grafischen Oberfläche des Imagers wählen wir nun die passenden Einstellungen:

{{% table %}}
| Kategorie | Auswahl / Details |
| :--- | :--- |
| **Gerät** | Raspberry Pi 4 |
| **Betriebssystem** | Raspberry Pi OS Lite (64-bit) |
| **Besonderheit** | Ohne grafische Oberfläche |
| **Speichermedium** | USB-Stick |
{{% /table %}}

Bevor der Schreibvorgang startet, passen wir über das Zahnrad-Symbol (Einstellungen) die individuellen Parameter an:

 {{% table %}} 

| Einstellung | Konfiguration / Wert |
| :--- | :--- |
| **Hostname** | raspServ |
| **Benutzer** | Benutzername & sicheres Passwort |
| **Schnittstellen** | SSH aktivieren (Headless-Betrieb) |
| **Lokalisierung** | Zeitzone & Tastaturlayout (de) |

{{% /table %}}

Die Abfrage zu „Raspberry Pi Connect“ beantworten wir mit Nein, da wir die Verwaltung lokal und "oldschool" über das Terminal durchführen möchten. Wer hier Nutzerkomfort vorzieht kann sich Raspberry Pi Connect aber problemlos mitinstallieren und mit dem Befehl  "rpi-connect sign-in" in einem Terminal vom PC die Verbindung zum Raspberry aufbauen.
Wir entscheiden uns wie gesagt für eine Installation ohne dieses Feature und bestätigen die Formatierung des Mediums.
Dies sollte nach einigen Sekunden bis zu wenigen Minuten abgeschlossen sein.

## Feste IP und erste Verbindung

Für einen Server ist eine gleichbleibende Erreichbarkeit essenziell. Bevor der Pi nun weiter Konfiguriert wird, empfiehlt es sich, im Router (z. B. in der Fritz!Box) eine feste IPv4-Adresse für das Gerät zu reservieren. Eine flüchtige IP-Adresse kann dazu führen, dass die Geräte später das Raspberry-Pi nicht korrekt identifizieren können und die spätere Pi-Hole konfiguration somit umgehen. {{< image src="/images/raspberry-pi/sameIp.png" alt="Festsetzen der Raspi IPv4-Adresse" caption="Festsetzen der Raspi IPv4-Adresse" width="300px" float="right" >}}
Dazu muss der PI an den Strom angeschlossen und per LAN-Kabel mit dem router verbunden werden. Die nötigen Einstellungen sind dann in der Fritz!box unter *Netzwerkeinstellungen* zu finden. Andere Routerhersteller haben eventuell abweichende Namen für die Einstellungen.  Es sollte allerdings ungefähr so heißen und aussehen wie hier abgebildet.

Sobald der Pi hochgefahren ist, öffnen wir am PC ein Terminal und bauen eine Secure-Shell-Verbindung auf:

```Bash
# verbinden mit Gerätenamen
ssh dein-nutzername@raspServ.local
# oder mit der festgelegten IPv4 Adresse
ssh dein-nutzername@<IPv4 Adresse im format www.xxx.yyy.zzz>
# Beispiel
ssh piUser@192.168.121.66
```

Beim ersten Verbindungsaufbau muss die Echtheit des Hosts durch die Eingabe von yes bestätigt werden. {{< image src="/images/raspberry-pi/sshFirst.png" alt="Erster erfolgreicher Verbindungsaufbau mit SHA 256 fingerprint" caption="Erster erfolgreicher Verbindungsaufbau mit SHA 256 fingerprint" width="600px" float="center" >}} Nach der Eingabe des Passworts bist du erfolgreich mit deinem Raspberry Pi verbunden. Die weitere Konfiguration von Pi-Hole und unbound stelle ich in den nächsten Blog-Einträgen vor. 

### Sinnvolle Idee zum Abschluss

Nachdem das Basissystem läuft, ist es ratsam, direkt ein System-Update in der SecureShell Verbindung durchzuführen, um auf dem neuesten Stand zu sein. Befehle variieren hier, je nach Betriebssystem und Distribution.

```Bash

sudo apt update && sudo apt upgrade -y
```

---

