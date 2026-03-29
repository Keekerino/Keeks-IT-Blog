---
title: "Terminalpersonalisierung"
date: 2026-03-29
tags: [Linux,CachyOS,Guide, Terminal]
draft: false
featured_image: "/images/Linux/linux-logo.svg"
description: "Farbakzente in CachyOS zum eigenen Vorteil nutzen"
---


Wer kennt das nicht beim Remote arbeiten: Man fährt nach getaner Arbeit wie immer über das Terminal die  Workstation per Befehl  herunter und wundert sich , wieso sich die Gehäuselüfter nicht aufhören zu drehen.

<!--more-->

Erst die Panik in der Stimme des dich anrufenden Kollegen und die nicht enden wollenden Drehgeräusche  der Kühlung bringen dich langsam zu einer Einsicht, die dir nun ebenfalls verbal bestätigt wird: Du hast nicht deinen PC, sondern  den Datenserver, an dem du heute Routinearbeiten durchführen solltest herunter gefahren.

Um solche Peinlichkeiten  in Zukunft möglichst zu vermeiden bietet sich eine visuell Abgrenzung durch Personalisierung des  Eingabe-Prompts und Terminals an. Dies kann man z.B. durch Ändern  des Hintergrundes in den Terminaleinstellungen der grafischen Oberfläche durchführen (Terminal1.png, Terminal2.png, TerminaNew.png)

{{< notice Grafisch >}}



In den Terminaleinstellungen der GUI(Grpahical User Interface) eine markante Hintergrundfarbe für Server-Verbindungen wählen.


{{< expand Ansicht >}}
{{< image src="/images/29.03.26/Terminal1.png" alt="Drop-Down Menü - Neues Profil erstellen" caption="" width="800px" >}}
{{< image src="/images/29.03.26/Terminal2.png" alt="Erscheinungsbild auswählen" caption="" width="800px" >}}
{{< image src="/images/29.03.26/TerminalNew.png" alt="Verändertes Erscheinungsbild" caption="" width="800px" >}}
{{< /expand >}}

{{< /notice >}}


{{< notice "Über die Konfiguration" >}}
In der Datei *~/.bashrc* lässt sich der Prompt *(PS1)* individuell anpassen. Diese Datei kann mit dem favorisierten Editor geöffnet werden. Hier am Beispiel für Nano.
```bash
    sudo nano ~/.bashrc
```
{{< expand Ansicht >}}
{{< image src="/images/29.03.26/Nano1.png" alt="Inhalt der Datei ~/bashrc" caption="" width="800px" >}}
{{< /expand >}}

Ein Beispiel für einen blauen Prompt:
`PS1='\[\e[0;34m\]\u@\h:\W\$\[\e[0;39m\]'`

*(Hinweis: Je nach Linux-Distribution können die Pfade variieren.)*

{{< /notice >}}

**Ein typischer Fallstrick:** Besonders beim Kopieren von Befehlen aus Guides oder von KIs werden oft unsichtbare Steuerzeichen oder falsche Formatierungen (wie ISO-8859-1) mitkopiert. Das führt zu Fehlermeldungen beim Start des Terminals. Tipp: Befehle immer erst in einen einfachen Texteditor kopieren oder manuell tippen, um „sauberen“ Code zu garantieren.