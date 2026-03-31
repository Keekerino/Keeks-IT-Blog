---
title: "Terminalpersonalisierung"
date: 2026-03-29
tags: [Linux,CachyOS,Guide, Terminal]
draft: false
featured_image: "/images/Linux/linux-logo.svg"
description: "Farbakzente in CachyOS zum eigenen Vorteil nutzen"
---

{{< notice "Wer kennt das nicht beim Remote arbeiten" >}}
Man fährt nach getaner Arbeit wie immer über das Terminal die  Workstation per Befehl  herunter und wundert sich , wieso sich die Gehäuselüfter nicht aufhören zu drehen.


<!--more-->

Erst die Panik in der Stimme des dich anrufenden Kollegen und die nicht enden wollenden Drehgeräusche  der Kühlung bringen dich langsam zu einer Einsicht, die dir nun ebenfalls verbal bestätigt wird: Du hast nicht deinen PC, sondern  den Datenserver, an dem du heute Routinearbeiten durchführen solltest herunter gefahren.

Um solche Peinlichkeiten  in Zukunft möglichst zu vermeiden bietet sich eine visuell Abgrenzung durch Personalisierung des  Eingabe-Prompts und Terminals an. Dies kann man z.B. durch Ändern  des Hintergrundes in den Terminaleinstellungen der grafischen Oberfläche durchführen. Diese Methoden sollten nahezu identisch auf anderen Linux Distributionen durchführbar sein.

{{< /notice >}}

{{< notice "Konfiguration in der grafischen Oberfläche" >}}

In den Terminaleinstellungen der GUI(Graphical User Interface) eine markante Hintergrundfarbe für Server-Verbindungen wählen.

{{< expand "Bilderreihe der Menüs in CachyOS" >}}
{{< image src="/images/29.03.26/Terminal1.png" alt="Drop-Down Menü - Neues Profil erstellen" caption="" width="400px" >}}
{{< image src="/images/29.03.26/Terminal2.png" alt="Erscheinungsbild auswählen" caption="" width="400px" >}}
{{< image src="/images/29.03.26/TerminalNew.png" alt="Verändertes Erscheinungsbild des Terminals nach Konfiguration" caption="" width="400px" >}}
{{< /expand >}}

{{< /notice >}}


{{< notice "Über die Konfigurationsdatei des Users" >}}
In der Datei *~/.bashrc* lässt sich der Prompt *(PS1)* individuell anpassen. Diese Datei kann mit dem favorisierten Editor geöffnet werden. Hier am Beispiel für Nano.
```bash
    sudo nano ~/.bashrc
```
{{< expand "Inhalt von ~/.bashrc" >}}
{{< image src="/images/29.03.26/Nano1.png" alt="Inhalt der Datei ~/.bashrc" caption="" width="400px" >}}
{{< /expand >}}

Ein Beispiel für einen blauen Prompt:
`PS1='\[\e[0;34m\]\u@\h:\W\$\[\e[0;39m\]'`

*(Hinweis: Je nach Linux-Distribution können die Pfade variieren.)*

{{< /notice >}}

**Ein typischer Fallstrick:** Besonders beim Kopieren von Befehlen aus Guides (z.B. Dieser Blog!) oder von KIs werden oft unsichtbare Steuerzeichen oder falsche Formatierungen (wie ISO-8859-1) mitkopiert. Das führt zu Fehlermeldungen beim Start des Terminals. Tipp: Befehle immer erst in einen einfachen Texteditor kopieren oder manuell tippen, um „sauberen“ Code zu garantieren. In Fällen bei denen ganze Scripte von Steuerzeichen bereinigt werden müssen bieten sich wiederrum kurze Befehle im Terminal an. 

{{< expand "Dieser Code hier löscht z.B. alle ASCII Steuerzeichenin einer Datei, behält aber Zeilenumbrüche (\n) und Tabulatoren (\t) bei" >}}

```bash
sed 's/[[:cntrl:]]//g' input.txt > output.txt
```
{{< /expand >}}