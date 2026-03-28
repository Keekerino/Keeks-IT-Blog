---
title: "Secure Shell Unter CachyOS"
date: 2026-03-28T09:24:58Z
draft: true
tags: []
featured_image: "/images/Linux/CachyOS_Logo.svg"
description: ""
---

{{< notice "SSH unter Arch Linux Konfigurieren - Eine Kurzanleitung" >}}
Einrichten von SecureShell, lokal unter der Arch-Linix Distribution CachyOS. Diese Kurzanleitung sollte bei jeder Arc-Linux Variante funktional sein.

**Hinweis:** Diese Konfiguration ist nur für lokale Netzwerke geeignet. Sobald man das Netz verlassen will, oder von außerhaLb zugreifen möchte, sollten weitere Sicherheitsvorkehrungen getroffen werden (SSH-Keys, andere Ports, Fail2Ban,..).

{{< /notice >}}

<!--more-->

### Grundkonfiguration
Um SSH auf einem Arch-Linux-System, wie CachyOS eines ist, zu aktivieren, wie folgt vorgehen
{{< notice >}}
1. Terminal öffnen ( Strg + Alt + t )
2. OPENSSH installieren über Terminal
    ```bash
    sudo pacman -S openssh
    ```
3. SSH-Dienst starten
    ```bash
    sudo systemctl start sshd
    ```
4. Autostart des Dienstes bei Systemstart aktivieren
    ```bash
    sudo systemctl enable sshd
    ```
{{< /notice >}}

### Firewall-Einstellung

1. Falls eine Firewall genutzt wird (dringend empfohlen!), muss noch eine Ausnahme für den Zugriff auf TCP Port 22 erstellt werden. Sollte eine andere Firewall als UFW genutzt werden, muss das Kommando entsprechend angepasst werden.
```bash
    sudo ufw allow 22/tcp
```
2. Beispiel für eine Firewalld-Konfiguration
```bash
    sudo firewall-cmd --permanent --add-service=ssh
    sudo firewall-cmd --reload
```

### Verbindungsherstellung

Nun kann man sich von einem anderen Rechner, der sich im gleichen loaklen Netz befindet, verbinden. Dazu muss ein Terminal geöffnet werden und mit der richtigen IP-Adresse verbunden werden. Die IP-Adresse findet man so heraus:
```bash
    ip addr show
    # oder einfach
    ip a
```
Dort nach der Zeile, die mit **inet** beginnt suchen -> dahinter steht die IP-Adresse

Um nun die Verbindung von einem anderen Unix-basierten System zu starten folgendes in das Terminal eintippen und bestätigen. Dabei Benutzername durch den tatsächlichen Benutzernamen auf der Zielmaschine ersetzen und die eben ermittelte IP-Adresse hinter dem **@** einfügen.
Nach Aufforderungsdialog Passwort eingeben.
```bash
    ssh benutzername@ip-adresse
```

Das war's auch schon. Nun sollte der Zugriff per SSH eingerichtet sein.