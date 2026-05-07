---
title: "Lokalen Hugo Webserver Absichern"
date: 2026-05-07T09:11:08+02:00
draft: true
author: "Keeks"
description: "Kurzanleitung"
featured_image: "/images/generic/Keeks4.png"
tags: []
categories: []
---



{{< notice Titel2 >}}
Hier könnte deine Beschreibung stehen
{{< /notice >}}

<!--more-->


---


## A: mkcert auf CachyOS installieren und CA erstellen
mkcert automatisiert die Erstellung einer lokalen CA und der passenden Zertifikate.
Installation auf CachyOS:
Bash
sudo pacman -S mkcert

Lokale CA initialisieren:
Bashmkcert -install

Dies erstellt eine Stammzertifizierungsstelle auf der VM. Den Pfad zu dieser Datei finden Sie mit mkcert -CAROOT heraus (meist ~/.local/share/mkcert/rootCA.pem).

## B: Zertifikate für die VM generierenErmitteln Sie zuerst die IP-Adresse Ihrer VM (z. B. via ip addr). Angenommen, die IP ist 192.168.100.15.

### Zertifikat erstellen:

Wechseln Sie in Ihr Hugo-Projektverzeichnis und führen Sie aus:
Bash
mkcert localhost 127.0.0.1 192.168.100.15

Sie erhalten zwei Dateien: localhost+2.pem (Zertifikat) und localhost+2-key.pem (Schlüssel).
## C: Die Vertrauenskette auf Windows 11 herstellen (Wichtigster Schritt)
Damit Chrome, Edge oder Firefox auf Windows nicht mehr warnen, muss Windows die rootCA.pem von der CachyOS-VM kennen.

### Datei kopieren:
Kopieren Sie die Datei rootCA.pem (Pfad siehe Schritt A.2) von der VM auf Ihren Windows-Wirt.
### In Windows importieren:
Rechtsklick auf rootCA.pem -> Zertifikat installieren.
Speicherort: Lokaler Computer.
Wählen Sie: Alle Zertifikate in folgendem Speicher speichern.
Klicken Sie auf "Durchsuchen" und wählen Sie: Vertrauenswürdige Stammzertifizierungsstellen.
Fertigstellen.


WICHTIG FÜR HUGO : -gitignore Datei aktualisieren, damit Zertifikate nicht auf gitHub geladen werden. Dazu folgenden Eitnrag in .gitignore vornehmen: 
# Zertifikate und Schlüssel
localhost+2-key.pem
localhost+2.pem

## Proxy-Ausnahme
Falls ein Proxy im Netzwerk verwendet wird, muss dieser in den Ausnahmen der Proxyverbindungen hinzugefügt werden. (nötige schritte hier sporadisch aufzeigen)

Schritt D: Hugo Server mit TLS startenIn Ihrem Code - OSS (VS Code) Terminal starten Sie Hugo nun mit den Pfaden zu den in Schritt B erstellten Dateien:Bashhugo server \
  --tlsCertFile localhost+2.pem \
  --tlsKeyFile localhost+2-key.pem \
  --bind 0.0.0.0 \
  --baseURL https://192.168.100.15 \
  --appendPort=true

3. Logische Herleitung der SicherheitKomponenteFunktion im ProzessRoot CA (rootCA.pem)Der "Anker". Wenn Windows dieser Datei vertraut, vertraut es allem, was damit unterschrieben wurde.End-Entity Cert (.pem)Das eigentliche Ausweisdokument für Ihren Hugo-Server, gültig für die spezifische IP.Private Key (-key.pem)Der geheime Schlüssel, mit dem Hugo beweist, dass es der rechtmäßige Besitzer des Zertifikats ist.4. Besonderheit bei "Code - OSS" (VS Code)Da Sie Hugo innerhalb von Code - OSS starten, stellen Sie sicher, dass die Dateipfade zu den .pem-Dateien korrekt sind. Wenn Sie die Dateien im Wurzelverzeichnis Ihres Hugo-Projekts abgelegt haben, reicht der Dateiname. Falls nicht, nutzen Sie absolute Pfade.Fazit für Ihre FortbildungDieses Verfahren entspricht exakt dem, was große Firmen intern machen (Unternehmens-CA). Für den späteren Schritt ins Internet (OpenWebUI) werden Sie dieses Prinzip beibehalten, aber die Zertifikatserstellung an Let's Encrypt delegieren, da deren "Root CA" bereits in jedem Browser der Welt vorinstalliert ist.Wenn Sie nun den Browser auf Windows öffnen und https://192.168.100.15:1313 eingeben, sollte das Schloss-Symbol grün/geschlossen sein.