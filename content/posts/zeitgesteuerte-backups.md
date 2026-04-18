---
title: "Zeitgesteuerte Backups  auf dem RaspberryPi"
date: 2026-04-18T08:33:13+02:00
draft: true
author: "Keeks"
description: "Cronjobs für Automatisierung nutzen"
featured_image: "/images/generic/Raspberry_Pi_Logo.svg"
tags: [Backup, RaspberryPi, Projekte]
categories: []
---



{{< notice "" >}}

Der RaspberryPi ist schon ein tolles Gerät. Damit man auch lange freude daran hat und im Notfall seine Daten gesichert hat sehen wir uns hier heute mal die Möglichkeit von inkrementellen Backups mittels *rsync* an.

{{< /notice >}}

 <!--more-->

In der IT gibt es zwei Arten von Menschen: Diejenigen, die bereits Daten verloren haben, und diejenigen, denen es noch bevorsteht. 

Da unser Betriebssystem in diesem Setup auf dem USB-Stick läuft, wird die SD-Karte kaum noch durch Schreibzugriffe belastet. Sie dient nur noch als ruhiger Lagerplatz für unsere rsync-Sicherungen und hält dadurch deutlich länger.

Empfehlungen aus dem Netz zum Nutzen des Repositorys Piclone für das Klonen des Betriebssystems eignen sich primär für das Kopieren auf eine externe SD-Karte. Aus diesem Grund – und auch, weil ich gerne ein inkrementelles Backup haben möchte – nutze ich das vielseitige Tool rsync. Zur besseren Verwaltung automatisieren wir das Ganze mit einem Script und den zeitgesteuerten Aufgaben in Linux: *Cronjobs*.

Rsync soll das Backup der wichtigsten Systemdaten in einen Ordner meines [SAMBA-Servers]({{< ref "posts/SAMBA-Dateiserver-Kurzanleitung.md" >}}) auf der integrierten SD-Karte schreiben. Dazu erstelle ich einen neuen Ordner unter */mnt/dein_pfad* und übergebe den Besitz an root, da auch das Script mit Root-Rechten ausgeführt wird. Wobei der Pfad natürlich auf die persönlichen Bedürfnisse angepasst werden sollte und hier nur beispielhaft steht.

**Sicherheitstipp**: In sensiblen Netzwerken empfiehlt es sich, den Backup-Ordner zu verstecken (indem man den Namen mit einem Punkt maskiert, z.B. *.backup*) oder den Zugriff in der Samba-Konfiguration explizit einzuschränken.

```Bash
sudo mkdir /mnt/dein_pfad
sudo chown root:root /mnt/dein_pfad/
```

# Script erstellen

Der Befehl rsync in Kombination mit Scripts

Der Hauptbefehl mit rsync wird mit dem absoluten Pfad angegeben, um auch in minimalistischen Betriebssystemen das Tool korrekt anzusprechen. Die Flags erkläre ich hier kurz:

{{< expand >}}

| **Flag** | **Beschreibung** |
| :--- | :--- |
| **-a (archive)** | Der wichtigste Schalter. Er sorgt dafür, dass fast alles erhalten bleibt: Unterverzeichnisse, Dateiberechtigungen, Zeitstempel und symbolische Verknüpfungen.
| **-x (one-file-system)** |  Dieser Parameter weist rsync an, auf dem aktuellen Dateisystem zu bleiben. Da die SD-Karte unter /mnt/ eingehängt ist, würde rsync ohne dieses Flag versuchen, die SD-Karte in sich selbst zu kopieren (Endlosschleife).
|**-H, -A, -X** | sichern Hard-Links, Zugriffssteuerungslisten (ACLs) und erweiterte Dateiattribute|
| **- -delete** | Dieser Befehl macht das Backup zu einem echten Spiegelbild. Dateien, die auf dem USB-Stick gelöscht wurden, werden auch im Backup entfernt.|

{{< /expand >}}

**Warum schließen wir so viel mit (--exclude) aus?**


Ein laufendes Linux-System enthält Ordner, die keine echten Dateien auf der Festplatte sind, sondern virtuelle Schnittstellen zum Kernel oder zum Arbeitsspeicher (wie /dev, /proc, /sys). Würden wir diese mitkopieren, käme es zu Fehlern oder riesigen, unbrauchbaren Datenmengen. Auch unseren freigegebenen Backup-Ordner für den [SAMBA-Server]({{< ref "posts/SAMBA-Dateiserver-Kurzanleitung.md" >}}) schließen wir aus (/mnt/* und /dein_pfad/*), damit rsync nicht versucht, das Backup in einer Endlosschleife in sich selbst zu sichern.
Inhalt des Scripts mit E-Mail

Dies sind zwei Varianten eines simplen Scripts, welches in der ersten Konfiguration auch Meldungen über Erfolg oder Misserfolg des Backups an den konfigurierten E-Mail-Account oder eine andere App weiterleiten kann. Die Konfiguration eines dafür benötigten App-Passworts und das Einrichten des Dienstes habe ich [hier übersichtlich erklärt]({{< ref "posts/mail-transfer-agent-nutzen.md" >}}).


## Inhalt des Scripts mit E-Mail

```bash
#!/bin/bash
# Backup ausführen
/usr/bin/rsync -axHAX --delete --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/dein_pfad/*,/lost+found} / /mnt/dein_pfad

# Prüfen ob rsync erfolgreich war (Exit Code 0)
if [ $? -eq 0 ]; then
    echo "Backup vom $(date) war erfolgreich." | mail -s "Backup OK: dein_gerätename" deine-email@beispiel.de
else
    echo "Backup vom $(date) ist FEHLGESCHLAGEN!" | mail -s "Backup FEHLER: dein_gerätename" deine-email@beispiel.de
fi
```

## Hier alternativ ohne eine E-Mail Zustellung

```bash
#!/bin/bash
# Backup ausführen
/usr/bin/rsync -axHAX --delete --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/dein_pfad/*,/lost+found} / /mnt/dein_pfad

# Prüfen ob rsync erfolgreich war (Exit Code 0)
if [ $? -eq 0 ]; then
    wall "Backup vom $(date) war erfolgreich."
else
    wall "Backup vom $(date) ist FEHLGESCHLAGEN!"
fi
```

## Script ausführbar machen (X-Recht vergeben)

```bash
sudo chmod +x /usr/local/bin/pi_backup.sh
```



## Cronjob anpassen

Die Verwaltung der zeitgesteuerten Aufträge wird in Linux über die sogenannten Cronjobs erledigt. Zuerst sollte man prüfen, ob der Dienst aktiv ist:

```Bash
sudo systemctl status cron
```

Sollte er nicht aktiv sein, starten wir ihn mit:

```Bash
sudo systemctl enable --now cron
```

Dann öffnen wir die Crontab-Konfigurationsdatei und tragen unseren Cronjob ein:

```Bash
sudo crontab -e
```

Eintrag in der Crontab-Konfigurationsdatei:

```Bash
0 3 * * 0 /usr/local/bin/pi_backup.sh
```
Die interaktive Web-Anwendung namens [Crontab-Guru](https://crontab.guru/) eignet sich hier gut zum Visualisieren und besseren Verständnis sowie dem Planen der Parameter.
Hier eingestellt ist ein Backup der Daten an jedem Sonntag um 3:00 Uhr.


# Funktionstest

Nun bleibt noch ein kurzer Test des Installationsscripts. Dazu kann *rsync* im *Dry-Run-Modus* ausgeführt werden mit dem *-n Flag*. Zusätzlich setzen wir noch das *Flag -v* für *verbose*, um eine detaillierte Ausgabe der durchgeführten Aktionen im Terminal zu erhalten, ohne dabei schon tatsächlich Daten zu schreiben.

```Bash
sudo rsync -anvxHAX --delete --exclude={/dev/*,/proc/*,/sys/*,/tmp/*,/run/*,/mnt/*,/dein_pfad/*,/lost+found} / /mnt/dein_pfad
```

Dies sollte eine lange Ausgabe auf dem Terminal produzieren und am Ende **KEINEN** Fehler auswerfen. Dann ist die Konfiguration gelungen.

Ab jetzt wird jeden Sonntag um 3:00 Uhr nachts automatisch ein Backup vom USB-Stick auf die interne SD-Karte des Raspberry Pi erstellt. Falls du auch eine [SAMBA-Server]({{< ref "posts/SAMBA-Dateiserver-Kurzanleitung.md" >}}) auf dem Server eingerichtet hast, kannst du nun von überall im Netzwerk auf deine Sicherungen zugreifen.

Dank der effizienten Funktionsweise von rsync werden ab dem zweiten Backup nur noch die Daten kopiert, die sich tatsächlich verändert haben. Das spart massiv Speicherplatz und schont die Ressourcen des Systems.