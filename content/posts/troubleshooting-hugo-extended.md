---
title: "Troubleshooting: Hugo Extended Setup"
date: 2026-03-19
draft: false
author: "Sebastian Failing"
description: "Wie ich Versionskonflikte beim Aufbau meines IT-Blogs gelöst habe."
categories: ["Projekte"]
tags: ["Hugo", "Linux", "Troubleshooting"]
---

## Projektübersicht
Ziel war der Aufbau eines IT-Blogs mit dem Static Side Generator Hugo in einem GitHub Codespace.

## Die Herausforderung
Die Standard-Installation von Hugo im Codespace war veraltet. Das Theme "Ananke" benötigt jedoch die **Extended-Version**, um CSS-Dateien zu verarbeiten. Ein Update der integriereten Hugo-Version mit get-apt liefert keine zufriedenstellenden Ergebnisse.

## Meine Herangehensweise

### 1. **Analyse:** 
* Identifikation der benötigten Version (0.146.0 Extended).
* Der Fehler äußerte sich durch folgenden Abbruch beim Start des Servers:
`Error: failed to transform "ananke/css/main.css" (text/x-scss): this feature is not available in your current Hugo version`
* Mit dem Befehl `hugo version` wurde festgestellt, dass eine Standard-Version installiert war. Für SCSS-Support wird zwingend die **Extended-Version** benötigt. Da im GitHub Codespace keine Administratorrechte für `apt-get` vorlagen, war eine manuelle Installation der Binary erforderlich.

### 2. **Installation:** 
* Manueller Download der Binary von GitHub und Speichern in Codespace
```bash
$ wget https://github.com/gohugoio/hugo/releases/download/v0.146.0/hugo_extended_0.146.0_linux-amd64.tar.gz
```
* Das tar-Archiv entpacken
```bash
$ tar -xvf hugo_extended_0.146.0_linux-amd64.tar.gz hugo
```

### 3. **Ausführung:**
* Start des Servers mit ab jetzt aus dem Hauptverzeichnis heraus mit 
```bash
./hugo server
```

## Fazit
Es ist grundlegened hilfreich, wenn man sich im Vorraus im Klaren darüber ist, welche version man benötigt um Abhängigkeitskonflikte direkt zu vermeiden. Das Systematische analysieren der Fehlerausgabe ist hierbei das wichtigste Tool, um zum Ziel zu kommen. Systemrepositories können veraltet sein, die falsche Version liefern, oder man besitzt nicht genügend Rechte im Zielsystem, ws vor Allem bei online Platformen häufig der Fall sein kann. Eine Manuelle Installation der benötigten Software umsetzen zu können ist hierbei besonders wichtig.