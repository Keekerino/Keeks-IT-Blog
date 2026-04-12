---
title: "Mail Transfer Agent für Raspberry"
date: 2026-04-12T11:58:35+02:00
draft: true
author: "Keeks"
description: "Überprüfen des Erfolgs von Backups mithilfe msmtp"
featured_image: "/images/generic/Raspberry_Pi_Logo.svg"
tags: [RaspberryPi, Automation, DIY, Projekte]
categories: []
---



{{< notice "E-Mail Benachrichtigungen des Pi's Konfigurieren für Benachrichtigungen von z.B. erfolgreich durchgeführten Backups." >}}
Damit der Raspberry Pi E-Mails senden kann, wird ein "MTA" (Mail Transfer Agent) benötigt. Ein leichtgewichtiger Standard für den Pi ist msmtp.
Um diesen wiederrum nutzen zu können, ist es zwingend notwendig ein sogennantes App-Passwort beim E-Mail provider einzurichten. Dies dient dem Schutz des Klarkennworts. Ich nutze diese Funktion mit meinem Zweit-Konto von google-mail inklusive einer weiterleitung auf meine Haupt-E-Mail Adresse eine Europäischen Anbieters.

<!--more-->

Dafür muss die 2-Faktor-Authentifizierung (2FA) deines Mail Accounts aktiviert sein. Dies ist in den Einstellungen des Google-Kontos unter "Sicherheit und Anmeldung -> 2-Faktor-Authentifizierung zu erledigen. Nach erfolgreicher Einrichtung von 2FA kann im Suchfeld der Konto-Optionen nach "App-Passwörter" gesucht werden -> oder scrollen, bis man die Option findet. {{< image src="/images/raspberry-pi/12.04.26-Backups/appPwd.png" alt="Beispiel für App-Passwörter Menü" caption="" width="300px" float="right" >}}
Dort kann nun ein App Name angegeben werden  . Das Passwort notieren und sicher speichern an einer Stell der Wahl für die integration im MTA und späteren Wiederverwendung


{{< /notice >}}


## Die Installation des Mail-Agenten

    sudo apt update
    sudo apt install msmtp msmtp-mta bsd-mailx

{{< image src="/images/raspberry-pi/12.04.26-Backups/msmtpAppArmor.png" alt="AppArmor Warnung" caption="AppArmor Warnung während Installation von msmtp" width="300px" float="right" >}}
Während der Installation eine Sicherheitsabfrage zu AppArmor,  die wir mit **yes** beantworten 



## Konfiguration des Mail Agenten

Jetzt muss eine Konfigurationsdatei für den Mail Agenten angelegt werden. Dafür benötigt werden die SMTP-Daten des E-Mail-Anbieters, z. B. Gmail, GMX, Outlook.
Manche Anbieter, wie z.B. ProtonMail unterstützen diese Funktion nicht.



    sudo nano /etc/msmtprc

### Beispielkonfigurationsdatei für Gmail


    defaults
    auth           on
    tls            on
    tls_starttls   on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt
    logfile        ~/.msmtp.log

    account        default
    host           smtp.gmail.com
    port           587
    from           deine-email@mail.com
    user           deine-email@mail.com
    password       dein-app-passwort



**Wichtige Sicherheitsbedenken: Nutze bei Gmail ein "App-Passwort", nicht dein normales Passwort**

**Außerdem ist es sehr zu empfehlen die Rechte an der Datei einzuschränken, so dass nur Root das Passwort auch lesen kann.**

    sudo chmod 600 /etc/msmtprc


## Prüfen der Konfiguration

Nun kann die Zustellung geprüft werden. Im Terminal diesen Befehl eingeben.
Dies sollte eine kurze E-Mail an das hinterlegte Postfach senden. 

    echo "Test-Backup erfolgreich" | mail -s "Pi-Backup Status" deine-email@mail.com


Somit wäre der Maildienst für Raspberry Pi eingerichtet und ab nun können in scripts automatisch Nachrichten an den Besitzer des E-Mail Kontos gesendet werden.
Dies ist besonders hilfreich zur Überprüfung des Ergfolges von automatischen Backups. Nun muss man nicht jedes mal die Logs selber prüfen, sondern wird benachrichtigt wenn das Backup gelungen ist, oder eben fehlschlägt. Sau praktisch!

---