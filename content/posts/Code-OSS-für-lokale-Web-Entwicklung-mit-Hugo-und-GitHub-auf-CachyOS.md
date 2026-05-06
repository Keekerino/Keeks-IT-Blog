---
title: "Kurzanleitung"
date: 2026-05-06T09:08:06+02:00
draft: false
author: "Keeks"
description: "Code OSS Für Lokale Web Entwicklung Mit Hugo Und GitHub Auf CachyOS"
featured_image: "/images/hugo/hugo-icon.svg"
tags: [Kurzanleitung,Code,Linux,CachyOS,GitHub,Hugo]
categories: []
---



{{< notice "Code - OSS für lokale Web-Entwicklung mit Hugo und GitHub auf CachyOS" >}}
Wenn man an verschiedenen Rechnern arbeitet und flexibel GitHub-Projekte verwalten möchte, ist es praktisch, die nötigen Schritte für eine erfolgreiche Installation der benötigten Software in einer Datei parat zu haben und nicht jedes Mal zu googeln. In diesem Fall geht es um die korrekte Einbindung des Frameworks Hugo in Code - OSS zum Verwalten der Themes eines Webprojektes.
{{< /notice >}}

<!--more-->




## Basispakete installieren
Zuerst ein Terminal öffnen (STRG + ALT + T) und Code - OSS sowie Git installieren. Für die Paketverwaltung unter CachyOS sind Root-Rechte erforderlich.

```bash
sudo pacman -S code git
```

## Git global konfigurieren
Um Repositories fehlerfrei zu nutzen und Commits korrekt zuzuordnen, müssen Benutzername und E-Mail-Adresse initial im System hinterlegt werden.

```Bash
git config --global user.name "Benutzername"
git config --global user.email "email@beispiel.de"
```

## Code - OSS starten und GitHub-Account verknüpfen
Nach der Installation Code - OSS auf die gewünschte Weise starten (über das Terminal oder per grafischer Oberfläche).  

Um Repositories einfach zu verwalten, ist es ratsam, sich direkt in Code - OSS mit dem GitHub-Account anzumelden. {{< image src="/images/05.06.26/account.png" alt="Kontaktplättchen auf korrekten Chips platzieren" caption="" width="300px" float="left" >}}  

Normalerweise erscheint beim ersten Start des Programms ein Dialog zur Verknüpfung des Accounts. 
Alternativ erfolgt die Anmeldung über das Personen-Symbol (Accounts) in der unteren linken Ecke.  


## Ein GitHub-Repository hinzufügen (Klonen)

Um das gewünschte Repository zu klonen, die Tastenkombination  

**STRG + Umschalt + P drücken**.

Anschließend **Git: Clone** eintippen und den Befehl aus dem Drop-Downm-Menü auswählen. Danach **Clone from GitHub** wählen. Code - OSS zeigt nun eine Liste der Repositories an. Das gewünschte Repository auswählen und in dem sich öffnenden Fensster einen lokalen Ordner auf dem CachyOS-System bestimmen, in dem das Projekt gespeichert werden soll. Abschließend mit **Open** bestätigen, um das Repository direkt in Code - OSS zu öffnen.

## Hugo installieren und Projekt initialisieren
In Code - OSS das integrierte Terminal öffnen (STRG + J oder STRG + Ö).

Hugo installieren:

```bash
sudo pacman -S hugo
```

Sicherstellen, dass alle Dateien im Theme-Ordner (Submodule) tatsächlich geladen werden:


```bash
git submodule update --init --recursive
```

Lokalen Entwicklungsserver starten:

```bash
# Dieser Befehl startet den Server normal
hugo server

# Alternativ:
# Starten mit "Drafts" aktiviert
hugo server -D

# Optional für verfügbarkeit im lokalen Netzwerk:
# (Firewall Ausnahme für Port 1313 TCP muss auf Host entsprechend gesetzt werden)
hugo server --bind 0.0.0.0 --baseURL http://DEINE_IP_ADDRESSE:DEINE_PORTNUMMER

```

Nach der Ausführung von *hugo server* *hugo server -D* ist der lokale Entwicklungsserver unter der Standardadresse [http://localhost:1313](http://localhost:1313) im Webbrowser vom Host aus erreichbar. 

Sollte man den Server im Netz exponiert haben, so kann man ihn nun durch eingabe mit dem Schema  

**http://IP-Adresse:Portummer**

von anderen Geräten im Netzwerk aus erreichen. 

Die lokale Entwicklungsumgebung unter CachyOS ist damit initial eingerichtet. Zukünftige Änderungen an den Projektdateien werden durch den laufenden Hugo-Server automatisch registriert und neu geladen. Die Versionierung des Codes und die Synchronisation mit dem GitHub-Repository erfolgen fortan wahlweise über die grafische Git-Integration von Code - OSS oder direkt über die Kommandozeile im integrierten Terminal.

---