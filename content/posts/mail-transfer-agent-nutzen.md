---
title: "Mail Transfer Agent für Raspberry"
date: 2026-04-14T11:58:35+02:00
draft: false
author: "Keeks"
description: "msmtp Dienst für Benachrichtigungen nutzen"
featured_image: "/images/smtp/SMTP-logo-color-2.svg"
tags: [RaspberryPi, Automation, DIY, Projekte, Mail, Linux]
categories: []
---



{{< notice "E-Mail Benachrichtigungen des Pi's Konfigurieren" >}}

Dein Raspberry Pi ist ein echtes Arbeitstier, aber leider auch ein ziemlich schweigsames. Er ackert im Hintergrund, und oft erfährst du erst von Problemen, wenn du dich mühsam per SSH einloggst. Warum also nicht den Spieß umdrehen und den Pi dazu bringen, dich bei Statusänderungen einfach anzumailen?


{{< /notice >}}

<!--more-->

Damit der Raspberry Pi E-Mails senden kann, wird ein MTA (Mail Transfer Agent) benötigt. Ein leichtgewichtiger Standard für den Pi ist **msmtp**.
Um diesen Dienst nutzen zu können, ist es notwendig ein sogennantes App-Passwort beim E-Mail provider einzurichten. Dieses ersetzt das eigentliche Kontopasswort und schützt es dadurch effektiv. In diesem Beispiel nutze ich diese Funktion mit einem Test-Mail-Konto von Google-Mail inklusive einer weiterleitung auf eine E-Mail Adresse eines Europäischen Anbieters.

Damit wiederrum msmtp auf dein Google-Konto zugreifen kann, muss die Zwei‑Faktor‑Authentifizierung (2FA) aktiviert sein. Dies ist in den Einstellungen des *Google-Kontos* unter

 **"Sicherheit und Anmeldung -> 2-Faktor-Authentifizierung** 

zu finden. Nach erfolgreicher Einrichtung von 2FA kann im Suchfeld der Konto-Optionen nach *App-Passwörter* {{< image src="/images/raspberry-pi/12.04.26-Backups/appPwd.png" alt="Beispiel für App-Passwörter Menü" caption="" width="300px" float="right" >}} gesucht werden, oder man scrollt, bis man die Option auf der Seite findet. 

Hier lässt sich nun ein App‑Name festlegen. Das generierte Passwort solltest du sicher notieren und aufbewahren, da es nur einmal angezeigt wird.




## Die Installation des Mail-Agenten

Nun zur eigentlichen Installation des Mail Agenten auf dem raspberry Pi. Wähle deine Bevorzugete Verbindungsmethode und öffne ein Terminal.

Zuerst **sudo apt update** zu nutzen, stellt sicher, dass Sie die neueste Version installieren, Abhängigkeiten korrekt aufgelöst werden und Fehler durch veraltete Verweise vermieden werden. Danach installieren wir die benötigten Pakete.

    
    sudo apt update

    sudo apt install msmtp msmtp-mta bsd-mailx

{{< image src="/images/raspberry-pi/12.04.26-Backups/msmtpAppArmor.png" alt="AppArmor Warnung" caption="AppArmor Warnung während Installation von msmtp" width="300px" float="right" >}}

Während der Installation der Pakete erscheint eine Sicherheitsabfrage des RaspberryPi bezüglich AppArmor
die wir mit **yes**  beantworten.





## Konfiguration des Mail Agenten

Jetzt muss eine Konfigurationsdatei für den Mail Agenten angelegt werden. Dafür benötigt werden die SMTP-Daten des E-Mail-Anbieters, z. B. Gmail, GMX, Outlook.
Einige Anbieter, wie z.B. ProtonMail unterstützen diese Funktion nicht.

Wir nutzen dafür wieder meinen Lieblingseditor Nano. Natürlich kann jeder andere, auf dem System vorhandene Editor auch genutzt werden. z.B. vim.

    sudo nano /etc/msmtprc

### Beispielkonfigurationsdatei für Gmail

```bash
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
```


Außerdem ist es **sehr empfehlenswert** und für manche Funktionen sogar **zwingend nötig** die Rechte an der Datei einzuschränken, so dass nur Root das Passwort auch lesen kann.


    sudo chmod 600 /etc/msmtprc


## Prüfen der Konfiguration

Nun kann die Zustellung geprüft werden. Im Terminal diesen Befehl eingeben.
Dies sollte eine kurze E-Mail an das hinterlegte Postfach senden. 

    echo "Test-Backup erfolgreich" | mail -s "Pi-Backup Status" deine-email@mail.com

Sollte keine Mail ankommen, prüfe die Log-Datei mit
    
    sudo journalctl -f msmtp

Damit ist der Mail-Dienst auf deinem Raspberry Pi startklar. Ab jetzt können deine Scripte dich proaktiv über alles informieren, was im System passiert – und zwar mit genau den Details, die du brauchst.

Für mich ist das vor allem bei automatisierten Backups ein absoluter Gamechanger. Statt mich mühsam durch kryptische Log-Dateien zu wühlen, erhalte ich nun eine maßgeschneiderte Statusmeldung direkt in mein Posteingang. Ob alles glattgelaufen ist oder es irgendwo hakt (inklusive komplexer Fehlerberichte), weiß ich jetzt sofort.

Wie genau ich diesen Mail-Versand in mein Backup-Script eingebunden habe, zeige ich dir in einem kommenden Post. Eines vorab: Es ist einfach sau praktisch!

---