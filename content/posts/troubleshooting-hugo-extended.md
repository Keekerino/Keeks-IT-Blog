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
Die Standard-Installation von Hugo im Codespace war veraltet. Das Theme "Ananke" benötigt jedoch die **Extended-Version**, um CSS-Dateien zu verarbeiten. Ein Update mit get-apt liefert keine Ergebnisse.

## Meine Lösungsschritte
1. **Analyse:** Identifikation der benötigten Version (0.146.0 Extended).
2. **Installation:** Manueller Download der Binary von GitHub.
3. **Ausführung:** Start des Servers mit `./hugo server`.

## Fazit
Hierbei habe ich gelernt, wie man binäre Abhängigkeiten in Linux-Umgebungen verwaltet. Und wie man Porgramme in abweichenden Pfaden außerhalb des /PATH ausführen kann