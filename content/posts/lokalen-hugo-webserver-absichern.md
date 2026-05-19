---
title: "Lokalen Hugo Webserver Absichern"
date: 2026-05-07T09:11:08+02:00
draft: true
author: "Keeks"
description: "Kurzanleitung"
featured_image: "/images/OfficialMSIcons/Microsoft Entra Verified ID color icon.svg"
tags: [Webserver, Hugo, Kurzanleitung, Security]
categories: []
---



{{< notice "Eine Kurzanleitung" >}}
Das Absichern des Netzwerkverkehrs stellt eine immer wichtigere Komponente in lokalen Netzen dar. Heute erläutere ich am Beispiel des Frameworks [Hugo](gohugo.io), wie man den Webserver des Frameworks mit TLS (Transport Layer Security) absichert. Dies entspricht Best Practices in internen Firmenstrukturen und ist in produktiven Netzen zwingend erforderlich. Außerdem gibt es noch einen kleinen Fun-Fact im Artikel. Es lohnt sich also reinzuschauen.
{{< /notice >}}

<!--more-->

---


## mkcert auf CachyOS installieren und CA erstellen

mkcert automatisiert die Erstellung einer lokalen CA (Certificate Authority) und der passenden Zertifikate.
Installation auf [CachyOS (Arch Linux Derivat)](https://cachyos.org):

```Bash
sudo pacman -S mkcert
```

Lokale CA initialisieren:

```Bash
mkcert -install
```
Dies erstellt eine Stammzertifizierungsstelle auf dem Betriebssystem. Den Pfad zu dieser Datei ermittelst du mit:

```Bash
mkcert -CAROOT 
```

(meist: ~/.local/share/mkcert/rootCA.pem).

## Zertifikate generieren 

Ermittle zuerst die IP-Adresse deines Systems (z. B. via *ip addr*). Angenommen, die IP ist 192.168.100.15.  

**Um eine reibungslose Funktionsweise zu garantieren ist es dringend zu empfehlen die IP-Adresse statisch zu setzen.** 

### Zertifikat erstellen:

Wechsel in dein Hugo-Projektverzeichnis und führe in einem Terminal aus:

```bash
mkcert localhost 127.0.0.1 192.168.100.15
```

Nun entstehen zwei Dateien: *localhost+2.pem (Zertifikat)* und *localhost+2-key.pem (Schlüssel)*.

## Die Vertrauenskette auf Windows 11 herstellen

Damit Browser wie Chrome, Edge oder Firefox auf Windows nicht mehr warnen, muss Windows die rootCA.pem von CachyOS kennen. 

{{< expand "Kleiner Fun Fact zum Edge Browser" >}}
Erst kürzlich gab es eine [Meldung auf heise.de](https://www.heise.de/news/Microsoft-Edge-Passwoerter-landen-als-Klartext-im-Speicher-11281407.html) zum Passwortmanager des Edge Browsers, der die Passwörter im Arbeitsspeicher ungeschützt vorhielt. Selbst mit minimalem Aufwand (einfacher Hex Editor und Verwendung der eingebauten Task-Manager-funktionen) konnten so die gesamten gespeicherten Passwörter ausgelesen werden. Dies wurde zwar stand [Mai 2026)](https://www.heise.de/news/Microsoft-Edge-Keine-Klartext-Passwoerter-mehr-im-Browserprozess-11296765.html) behoben, ist jedoch meines Erachtens nach immer noch kein Grund dem Edge Browser zu vertrauen. [Der norwegische Browser Vivaldi](https://vivaldi.com/de/) bietet sich als europäische Alternative an. Er kombiniert die Webkompabilität von der Chromium-Engine mit einem strikten Verzicht auf Tracker und einer Infrastruktur, die vollständig europäischen Datenstandards entspricht.
{{< /expand >}}

### Datei kopieren:

Kopiere die Datei rootCA.pem (Pfad siehe Schritt A.2) von CachyOS auf den Windows-Clienten.

### In Windows importieren:

*Rechtsklick auf rootCA.pem -> Zertifikat installieren.*  

*Speicherort:* Lokaler Computer.  

*Wähle:* Alle Zertifikate in folgendem Speicher speichern.  

*Klicke auf "Durchsuchen" und wähle:* Vertrauenswürdige Stammzertifizierungsstellen.  

*Fertigstellen.*


**WICHTIG FÜR HUGO:** .gitignore Datei aktualisieren, damit Zertifikate nicht auf gitHub geladen werden. Dazu folgenden Eintrag in .gitignore hinzufügen:

    localhost+2-key.pem  
    localhost+2.pem

## Proxy-Ausnahme

Falls ein Proxy im Netzwerk verwendet wird, muss die IP-Adresse des Webservers in den Ausnahmen der Proxyverbindungen hinzugefügt werden. In diesem Fall also *192.168.100.15*

## Firewall-Konfiguration (ufw)

Da auf dem System die Uncomplicated Firewall (ufw) aktiv ist, blockiert diese standardmäßig eingehende Verbindungen auf Ports, die nicht explizit freigegeben sind. Vor dem Start des Servers muss der TCP-Port 1313 geöffnet werden:

```Bash
sudo ufw allow 1313/tcp
```

## Hugo Server mit TLS starten
In Code - OSS (VS Code) nun ein Terminal aufrufen und  Hugo nun mit den Pfaden zu den erstellten Dateien starten:

```Bash
hugo server \
  --tlsCertFile localhost+2.pem \
  --tlsKeyFile localhost+2-key.pem \
  --bind 0.0.0.0 \
  --baseURL https://192.168.100.15 \
  --appendPort=true
```

Wenn du nun den Browser auf Windows öffnest und  
[https://192.168.100.15:1313](https://192.168.100.15:1313)  
eingibst, sollte das Schloss-Symbol des Browsers grün/geschlossen sein.

### Besonderheit bei "Code - OSS" (VS Code)

Da Hugo in diesem Beispiel innerhalb von Code - OSS gestartet wird, muss sichergestellt werden, dass die Dateipfade zu den *.pem-Dateien* korrekt sind. Wenn die Dateien im Wurzelverzeichnis des Hugo-Projekts liegen, reicht der Dateiname. Falls nicht, müssen absolute Pfade genutzt werden.

## Schluss und Aussicht

Dieses Verfahren entspricht exakt dem, was große Firmen intern umsetzen (Unternehmens-CA). Für den geplanten späteren Schritt meine lokale KI im Internet mit [OpenWebUI](https://openwebui.com) verfügbar zu machen werde ich dieses Prinzip beibehalten, aber die Zertifikatserstellung an [Let's Encrypt](https://letsencrypt.org) delegieren, da deren "Root CA" bereits in jedem Browser der Welt vorinstalliert ist. 

