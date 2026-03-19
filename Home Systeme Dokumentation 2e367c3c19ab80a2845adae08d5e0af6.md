# Home Systeme Dokumentation

# Windows Server - AD DS 2025 Dokumentation

## Struktur und Topologie

### Domänenname

test.lab

### Server

- RRAS Router Server als Gateway
    
    Windows Server 2022 Desktop
    
    Name: GT-01-SRV-V
    
    Externe IPv4: 192.168.178.2/24 
    
    Interne IP: 10.10.0.1/24
    
- 1. Domänencontroller
    
    Windows Server 2022 Desktop
    
    DC-01-SRV-V
    
    IPv4: 10.10.0.2/24
    
- 2. Domänencontroller
    
    Windows Server 2022 Core
    
    DC-02-SRV-V
    
    IPv4: 10.10.0.3/24
    

### Clienten

- Linux OpenSuse
    
    Suse-Clie-01-V
    
    IPV4: 10.10.0.6
    
    Nutzer: Lokalrolf/0000 Rolf Lokal/P@ssw0rd
    
- Windows 11 Client
    
    Win-Clie-01-V
    
    IPV4: 10.10.0.7
    
    Nutzer: Lokalrolf
    

### User & Groups

- **Domäne test.lab**
    - Administrator:
        
        S3bbl3r : P@ssw0rd
        
        DSRM : DeinPasswort123!
        
    - Nutzer:
        - Extern
            
            Ben Utzer : P@ssw0rd
            
            Karsten Knall : P@ssw0rd
            
    - Gruppen
        
        Reudnitz(Glob./Sich.)
        
        Shadow Password Group(Glob./Sich.)
        
    
    - **Lokal**
        
        FreeSeb : 2139x5319km@S
        
    

### OU-Struktur Erstentwurf (veraltet)

- Labor (Haupt-OU)
    - Administration (1)
        
        S3bbl3r
        
    - Benutzer (2)
        
        Marketing
        
        Vertrieb
        
        Extern
        
    - Server (3)
        - Domänencontroller
            
            DC-01-SRV-V
            
            DC-02-SRV-V
            
        - Member
            
            GT-01-SRV-P
            
    - Clients (4)
        
        Suse-Clie-01-V
        
        Win-Clie-01-V
        
    
    - Ressourcen (Drucker, Fileshares, etc.) (5)
    

## Vorgehensweise

- Windows Server unter Windows 11 mit Hyper-V installieren
    
    Achten auf die Konfiguration des virtuellen Switches. Bei Win11 übernimmt der Default Switch NAT und DHCP, bei WinServer nicht. 
    

Alle verfügbaren Updates Installieren und neu starten

Grafikkartentreiber (wenn nötig für zweiten Bildschirm) installieren 

- **Einmalig:** Installation **generalisieren** mit **sysprep** und **mode:vm →** dieser spezielle Parameter sollte nur verwendet werden, wenn die virtuelle Maschine auf dem gleichen Host verbleibt. ~~Als weitere Zusatz bietet sich /s /0 an für Herunterfahren in 0 Sekunden →~~  Gillt nur im eigenständigen CMDlet: shutdown
    
    ```powershell
    # **Wichtig: Den genauen Pfad von Sysprep angeben!**
    C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /vm /shutdown
    ```
    
- Computernamen vergeben
    
    GT-01-SRV-V
    
- IP-Konfiguration für Rechner vergeben
    
    Zuerst DC Statisch mit IP versorgen
    
    Virtuelle switche erstellen. Einmal intern (NATintern) und extern(NATextern)
    
    - ~~Eventuell nötige IP-FritzBox Konfiguration:~~
        
        ~~IP Adresse von GT-01-SRV-P im DNS-Rebind Schutz des Routers angeben~~
        
        ~~Eventuell auch den Verweis zum verwendeten DNS-Server ändern →~~  **prüfen** 
        
    
- Installieren von Remotedesktopdiensten mit RRAS  auf ~~Physischem~~  virtuellem Server

Achten auf: Secure Boot, TPM, Angemessene Anzahl an Prozessorkernen und RAM

## Einrichten der Virtuellen Maschinen mit sofortiger Prüfpunkterstellung nach Einrichtung.

**J**ede Maschine zuerst Updaten und Installation des Grafiktreibers überprüfen, sollte zweiter Bildschirm benötigt werden.
**PC-NAMEN ÄNDERN**

Deaktivieren der Automatischen Prüfpunkte nicht vergessen! -> nicht unendlich Platz auf Platte. 

Beim Konfigurieren der DNS-Server, muss in einer Domäne mit mehr als einem DC der erste Eintrag zwingend auf den zweiten DC mit DNS verweisen

- **DC-01-SRV-V**
    - **AD Domänendienste, DNS
    und  Weiterleitungen im DNS Konfigurieren (z.B.: 8.8.8.8 →google.dns)**
        
        ```jsx
        “Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools”
        ```
        
        ```powershell
        $password = ConvertTo-SecureString "DeinPasswort123!" -AsPlainText -Force
        
        Install-ADDSForest -DomainName "test.lab" -DomainNetbiosName "TEST" -InstallDns:$true -SafeModeAdministratorPassword $password -NoRebootOnCompletion:$false
        
        ```
        
        ![image.png](Home%20Systeme%20Dokumentation/image.png)
        
    - **Zeitserver auf “[pool.ntp.org](http://pool.ntp.org/)” umgelegt mit dem tool w32tm
    Möglichkeit: Konfigurieren der Zeitsynchronisierung per externer Quelle über die Befehlszeile für den PDC**
    Administrative Eingabeaufforderung
        
        ```powershell
        **# Einstellungen setzen
        w32tm.exe /config /syncfromflags:manual /manualpeerlist:131.107.13.100,0x8 /reliable:yes /update
        
        # Updaten der config
        w32tm.exe /config /update**
        ```
        
    - **Anlegen der OU mit Power Shell**
        
        ```powershell
        # 1. Haupt-OU Labor
        New-ADOrganizationalUnit -Name "Labor" -Path "DC=test,DC=lab"
        
        # Basispfad für die Unter-OUs
        $basePath = "OU=Labor,DC=test,DC=lab"
        
        # 2. Die 5 Haupt-Unter-OUs erstellen
        New-ADOrganizationalUnit -Name "Administration" -Path $basePath
        New-ADOrganizationalUnit -Name "Benutzer" -Path $basePath
        New-ADOrganizationalUnit -Name "Server" -Path $basePath
        New-ADOrganizationalUnit -Name "Clients" -Path $basePath
        New-ADOrganizationalUnit -Name "Ressourcen" -Path $basePath
        
        # 3. Unter-OUs für Benutzer
        $benutzerPath = "OU=Benutzer,OU=Labor,DC=test,DC=lab"
        "Marketing", "Vertrieb", "Extern" | ForEach-Object {
            New-ADOrganizationalUnit -Name $_ -Path $benutzerPath
        }
        
        # 4. Unter-OUs für Server
        $serverPath = "OU=Server,OU=Labor,DC=test,DC=lab"
        "Domänencontroller", "Member" | ForEach-Object {
            New-ADOrganizationalUnit -Name $_ -Path $serverPath
        }
        
        Write-Host "Deine angepasste OU-Struktur wurde erfolgreich erstellt!" -ForegroundColor Cyan
        ```
        
    
- **DC-02-SRV-V**
    - IPv4 Konfiguration über die PowerShell
        
        ```powershell
        #Neue IP-Konfig
        New-NetIPAddress -InterfaceIndex 6 -IPAddress 10.10.0.3 -PrefixLength 24 -DefaultGateway 10.10.0.1
        
        #DNS auf ersten DC setzen und als zweiten Loopback
        Set-DnsClientServerAddress -InterfaceIndex 6 -ServerAddresses ("10.10.0.2", "127.0.0.1")
        ```
        
    - AD Domänendienste , DNS über PS
    Das führt immer wieder zu Problemen. Hier genau drauf achten. Vor allem die Vergabe der **“Credentials”** ist sehr wichtigp
        
        ```powershell
        # 1. DSRM Passwort (für diesen neuen DC) festlegen
        
        $dsrmSafe = ConvertTo-SecureString "DeinPasswort123!" -AsPlainText -Force
        
        # 2. Den Befehl in einer einzigen Zeile ausführen
        Install-ADDSDomainController -DomainName "test.lab" -InstallDns:$true -Credential (Get-Credential) -SafeModeAdministratorPassword $dsrmSafe
        ```
        
        - Beim weiteren Installationsprizess gab es fehler. NAch rechereche und auslesen des fehler logs  konnte ich folgendes finden:
            
            ```powershell
             Get-Content C:\Windows\debug\dcpromo.log -Tail 50
            ```
            
            > "01/11/2026 00:36:42 [INFO] EVENTLOG (Error): NTDS Database / Sicherung : 2542
            > 
            
            > Vom Verzeichnisserver wurde festgestellt, dass die Datenbank ersetzt wurde. Dies ist ein
            > 
            
            > ungesicherter und nicht unterstützter Vorgang. Der Dienst wird beendet, bis das Problem
            > 
            
            > behoben ist.
            > 
            
            > Benutzeraktion:
            > 
            
            > Stellen Sie die vorherige Kopie der auf diesem Computer verwendeten Datenbank wieder her."
            > 
            
            > und
            > 
            
            > "
            > 
            
            > 01/11/2026 00:36:42 [INFO] EVENTLOG (Informational): NTDS General / Dienststeuerung : 1004
            > 
            
            > Die Active Directory-Domänendienste wurden heruntergefahren.
            > 
            
            > 01/11/2026 00:36:42 [INFO] Error - Bei der Installation der Active Directory-Domänendienste ist ein unbekannter Fehler
            > 
            
            > aufgetreten. (8200)
            > 
            
            > 01/11/2026 00:36:42 [INFO] Es wird versucht, den \Registry\Machine\System\CurrentControlSet\Services\NTDS-Registrierungsschlüssel wiederholt zu löschen (DeleteRoot=0).
            > 
            
            > 01/11/2026 00:36:42 [INFO] Es wird versucht, den \Registry\Machine\System\CurrentControlSet\Services\NTDS\Diagnostics-Registrierungsschlüssel wiederholt zu löschen (DeleteRoot=1)
            > 
        
        Das öffnen der Firewall Profile und das erhöhen der MTU, um Abgewiesene Pakete durchzulassen  brachte auch keinen sichtbaren Erfolg. Nun hängt die Installation beim Prüfen der Vorraussetzungen und nicht erst bei dem Schritt danach.
        Server erneut  von vorne aufsetzen und mich von Anfang an genauer auf die Details fokussieren.
        Vor Allem DNS Einträge prüfen. 
        
        Vielleicht gab es Probleme durch  ein falsches Anmeldenamensformat?
        Ich sollte das Down-Level Format ausprobieren:
        
        **test.lab\Administrator**
        
        Zu diesem Zweck erstell ich ein  Konto mit dem namen S3bbl3r und füge es den relevanten Administatorengruppen hinzu
        
        Erstmal alles Deinstallieren und die Verzeichnisse frei machen:
        
        ```powershell
        Uninstall-WindowsFeature AD-Domain-Services -Force
        
        # 2. Datenbank- und Log-Ordner physisch löschen
        Remove-Item -Path "C:\Windows\NTDS" -Recurse -Force -ErrorAction SilentlyContinue
        Remove-Item -Path "C:\Windows\SYSVOL" -Recurse -Force -ErrorAction SilentlyContinue
        
        # 3. Neustart erzwingen, um Registry-Sperren zu lösen
        Restart-Computer -Force
        ```
        
        Dann erneut installieren mit
        
        ```powershell
        #Dieser Einzeiler hat es dann gerockt.
        
        Install-ADDSDomainController -DomainName "test.lab" -InstallDns:$true -NoGlobalCatalog:$false -NoRebootOnCompletion:$true -Force:$true -SafeModeAdministratorPassword (ConvertTo-SecureString 'DeinPasswort123!' -AsPlainText -Force) -Credential (Get-Credential "test.lab\S3bbl3r") -Verbose
        ```
        
        Der DC musste vorher NICHT teil der Domäne sein und sollte es auch nicht. Er tritt über die Hochstufung zum DC der Domäne automatisch bei
        Die Anmeldung mit test.lab\S3bbl3r über den Befehl Get-Credential war die richtige Wahl - > Keine Sicherheitskonflikte mehr.
        
        ```powershell
        #Bericht über Replikation ergab 0/5 Fehlern
        repadmin /replsummary
        ```
        
        Abhängigkeiten für AD Installieren(Samba, Yast und co.) **Tutorial ansehen**
        
        In Domäne aufnehmen
        
    
- **Win-Clie-01-V**
    - Windows Installation normal über Hyper-V
        
        Nachträgliche Netzwerkkarteneinstellung während Installation durch drücken von Shift+F10 über get-netadapter und dann new-netipaddress und set-dnsclient
        
        ```powershell
        # Wieder mit Zwei Befehlen,
        # wie zuvor
        New-NetIPAddress -InterfaceIndex 6 -IPAddress 10.10.0.3 -PrefixLength 24 -DefaultGateway 10.10.0.1
        
        Set-DnsClientServerAddress -InterfaceIndex 6 -ServerAddresses ("10.10.0.2","10.10.03")
        ```
        
        Durch frühes joinen in Domäne leider nicht mehr möglich ein sysprep image zu erstellen.
        
        Neuinstallieren 
        
        ```powershell
        # Bereinigen des Update Caches 
        # nach Updatets
        Stop-Service wuauserv
        Remove-Item -Path C:\Windows\SoftwareDistribution\Download\* -Recurse -Force
        Start-Service wuauserv
        ```
        
        unnötige Funktionen deaktivieren 
        
        - Bei Installation Online Account umgehen mit PS Kommando .
        Das funktioniert nicht ohne vorher die Netzwerkkarte zu deaktivieren, was ich auch tue
            
            ```powershell
            OOBE/BYPASSNRO
            ```
            
        - Nach Anmeldung mit lokalem konto wird das Administratorkonto aktiviert und damit angemeldet. Dann wird der lokale Nutzer wieder gelöscht.
            
            ```powershell
            
            # 1. Das echte Administrator-Konto aktivieren
            net user administrator /active:yes
            
            # 2. Passwort vergeben
            net user administrator *
            ```
            
        - Sysprep durchführen über PS
            
            ```powershell
            & $env:SystemRoot\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown /mode:vm
            ```
            
            Nach dem Erstellen der sysprep muss der Prüfpunkt in Hyper-V gelöscht werden, damit die beiden Datendateien zusammengeführt werden und so eine einzelne Festplatte  entsteht. Diese kann nun beliebig kopiert werden
            
            Der PC wird nun noch in die Domäne aufgenommen und ist dann Einsatzbereit
            
        
- **Suse-Clie-01-V**
    
    Normal über Hyper-V installieren.
    Secure Boot muss deaktiviert werden.
    Es treten teilweise Probleme bei der Initialisierung der Tastatur auf. Auch Ersatztastaturen zeigen dieselben Fehler.
    
    Nach der Erstinitialisierung des Systems funktioniert die Tastatur.
    
    - Die Grundeinstellungen für IP und Hostname wurden über die grafische Oberfläche vorgenommen. Dabei Ping, DNS Konnektivität und Zeit (aufklappen) prüfen. Wenn Ping zur Domäne gelingt und der Name aufgelöst werden kann, weiter mit nächstem Schritt.
        
        ```powershell
        #Status der NIC -> "up" = gut
        ip link
        
        #Ip-Konfig abfragen
        ip addr
        
        # Ping (-c 4 sendet 4 Pings)
        ping -c 4 gateway
        ping 8.8.8.8
        ping hostname
        
        ping auf Domänencontroller
        
        # Auflösen der DC
        host test.lab
        
        #Zeitabweichungen bestimmen
        timedatectl
        ```
        
    - Nach der Vergabe der richtigen “Searchdomain” wird mit dem Tool “realm” der Ad Beitritt vorbereitet und dann der Domäne beigetreten
        
        ```powershell
        # Installieren der Abhängigkeiten
        
        sudo zypper install realmd adcli sssd sssd-ad
        
        # Überprüfen der Konnektivität und 
        # Anzeigen der zu installiernden Abhängikeiten
        # (Samba-client und sssd-tools müssen eventuell noch hinzugefügt werden)
        sudo realm discover test.lab
        
        # Der Join in die Domäne
        sudo realm join -U Administrator test.lab
        
        # Identitätsdienst nochmal "anschubsen"
        sudo systemctl enable --now sssd
        
        # Home Verzeichnis für Windows User erlauben
        sudo pam-config -a --mkhomedir
        ```
        
        - Hier noch eine kleine Anleitung, damit keine FQDNS beim Anmelden genutzt werden müssen:
            
            ```powershell
            # Konfigurationsdatei öffnen
            sudo nano /etc/sssd/sssd.conf
            
            # Zeile suchen
            use_fully_qualified_names = True
            
            # Ändern
            use_fully_qualified_names = False
            
            # Speichern mit Strg+O, Enter und Schließen mit Strg+X.
            # Dienst neu starten:
            ****sudo systemctl restart sssd
            ```
            
        
        ```powershell
        # Domänen-Admin zum lokalen Admin machen (optional)
        sudo usermod -aG wheel S3bbl3r
        ```
        
        - Firmenlogo per Netzwerkfreigabe Importieren für Hintergrundbild
            
            ```powershell
            # Verzeichnis erstellen
            sudo mkdir -p /usr/share/wallpapers/custom
            
            # Import starten über Netzwerkfreigabe
            sudo smbget smb://S3bbl3r@dc-01-srv-v/Pictures/backgroundGG.jpg -o /usr/share/wallpapers/custom/backgroundGG.png
            ```
            

## AD-DS Konfiguration

- **Wichtige Ports in Active-Directory (Firewall Ausnahmen)**
    
    
    | Dienst | Protokoll | Port |
    | --- | --- | --- |
    | LDAP | TCP/UDP | **389** |
    | LDAP über SSL (LDAPS) | TCP | **636** |
    | Global Catalog | TCP | **3268** |
    | Global Catalog über SSL | TCP | **3269** |
- ms-DS-MachineAccountQuota attribute sollte mit PowerShell auf Null gesetzt werden.  Ansonsten können normale Computer Accounts bis zu 10 Computer Objekte in AD anlegen.
    
    ```powershell
    
    # Prũfen des wertes ms-DS-MachineAccountQuota
    Get-ADObject -Identity ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
    
    # Wert auf Null setzen
    Set-ADDomain -Identity (Get-ADDomain).distinguishedname -Replace @{"ms-DS-MachineAccountQuota"="0"}
    
    ```
    

### Erstellen Gruppenrichtlinien Teil 1

Da die Gruppenrichtlinien für den Start- und Sperrbildschirm, welche ich unbedingt umsetzen möchte aus Gründen, nicht so ohne weiteres auf windows Pro wirken, verwende ich drei Registry Einträge mit Verweis unter Computerkonfiguration → Einstellungen → Windows Einstellungen → Registrierung wobei die Verweise zur Zieldatei auf die lokalen Verzeichnisse geändert werden müssen. 
Freigabe : \\DC-01-SRV-V\Pictures\backgroundGG.png

lokaler Pfad: %Systemroot%\Web\Screen\backgroundGG.png 

**Die Pfade für die Werte (Fraigabe und Speicherort) müssen eventuell geändert werden und sind hier nur symbolisch**

- **Registry Eintrag in 
SOFTWARE\Policies\Microsoft\Windows\Personalization und SOFTWARE\Microsoft\Windows\CurrentVersion\PersonalizationCSP**
    
    ![grafik.png](Home%20Systeme%20Dokumentation/grafik.png)
    
- **Zusäzlicher Registry Eintrag zum verhindern des Änderns durch Benutzer**
    
    ![grafik.png](Home%20Systeme%20Dokumentation/grafik%201.png)
    
- **Nun wird über Computerkonfiguration → Einstellungen→ Windows-Einstellungen→ Dateien die Datei über die Bereitstellung lokal verfügbar gemacht**
    
    ![image.png](Home%20Systeme%20Dokumentation/image%201.png)
    
- **In den Eigenschaften der Datei noch “Element Entfernen” auswählen, dies ändert die Aktion beim Ausführen im ersten Reiter auf “Ersetzen”**
    
    ![image.png](Home%20Systeme%20Dokumentation/image%202.png)
    
- **Optionale Gruppenrichtlinienberechtigungen setzen:**
    
    Danach wird die in den Gruppenrichtlinieneinstellungen die Sicherheitsfilterung überprüft und bei Bedarf noch Authentifizierte Benutzer (umfasst alle Domänenbenutzer  UND Computer) hinzugefügt, um den Zugriff zu sichern. Im Fenster Sicherheitsfilter auf der ersten Seite und Deligierung auf dem letzten Reiter des GPO’s werden nun noch die Domänencomputer hinzugefügt mit der Berechtigung **lesen .**
    
     **Im Home Lab war das nicht nötig und auch in der Schule zeigte das nicht den gewünschten Effekt. also ist diese Einstellung erstmal mit vorsicht zu genießen.**
    *(Kontext: Seit einem Sicherheitsupdate  (MS16-072) verlangt Windows, dass der Computer im Systemkontext die GPO lesen darf.)*
    
    ![grafik.png](Home%20Systeme%20Dokumentation/grafik%202.png)
    
    ![grafik.png](Home%20Systeme%20Dokumentation/grafik%203.png)
    

### ERFOLG!

![image.png](Home%20Systeme%20Dokumentation/image%203.png)

![image.png](Home%20Systeme%20Dokumentation/image%204.png)

### gMSa’s (Group Managed Service Accounts) Erstellen

- **Voraussetzungen und Vorgehensweise:**
    - Admin muss Mitglied in “Organisations-Admins” sein
    1. KDS Root Key erstellen
    2. Sicherheitsgruppe anlegen
    3. gMSA erstellen
    4. gMSA auf Servern installieren
    5. Dienst auf gMSA umstellen
    6. SPN prüfen
    7. SPN ggf. manuell setzen
- **Fehlersuche**
    - Der Container “Key Distribution Service” muss im AD vorhanden sein. Erstellung erfolgt normalerweise automatisch. Alternativ über ADSI  Editor → Namenskontext Konfiguration Pfad: CN = Services → hier “CN = Key Distribution Service” erstellen.
        
        ```powershell
        # Powershell Erweiterung erneut korrekt im system registrieren 
        Install-WindowsFeature RSAT-AD-PowerShell
        
        # Reparieren der lokalen Windows Datei 
        # (Reparatur des Windows-Komponenten-Store)
        dism /online /cleanup-image /restorehealth
        # Best PRactices Empfihelt die verwendung von sfc nach  dism
        sfc /scannow
        
        # Danach frisches laden von Key Distribution Service Module
        Import-Module Kds -Force
        
        # Überprüfen des tatsächlichen Pfades mithilfe der PS
        # Suchen des Containers direkt über seinen Namen in der Configuration-Partition
        >> $target = Get-ADObject -Filter "Name -eq 'Key Distribution Service'" -SearchBase (Get-ADRootDSE).configurationNamingContext
        >> $target.DistinguishedName
        
        # Das Objekt nun in AD hinzufügen
        New-ADObject -Name "Key Distribution Service" -Type "container" -Path "CN=Services,$((Get-ADRootDSE).configurationNamingContext)"
        ```
        
        - Erkannte Fehler
            1. CN = Configuration war doppelt vorhanden, weswegen Windows nie den korrekten  Pfad finden konnte
            2. Fehlercode 0x80070020 sperrt eine .dll Datei des KDS Moduls physisch auf dem Server → der bestehende lock ließ sich nicht umgehen
            3. Der Prozess scheint blockiert und lässt sich nicht “unblocken”
            4. Sharing Violations blocken den Zugriff bereits auf OS-Ebene
        
- **Fazit**
    
    Da die verzweigten PowerShell Befehle nach mehrstündiger Fehlersuche keinen Erfolg bringen, wir das Projekt erstmal pausiert, bis ein tieferes Verständnis von AD vorhanden ist und die Domäne neu und sauber aufgesetzt wurde.
    

# **CachyOS Virtuelle Maschine 15.02.2026**

<aside>
💡

**Hyper-V-Konfiguration**

Secure Boot: Aus

Virtuelle Maschine der zweiten Generation erstellen 

Arbeitsspeicher: 4 GB dynamisch

Logische Prozessoren: 4

Festplatte: 45 GB GPT-Partitionstabelle im  Btrfs-Format. 

Netzwerk: Default Switch mit DHCP-Konfig. 

Automatische Prüfpunkte: Deaktiviert

**Installationskonfiguration**

Bootloader Konfiguration: GRUB

Desktopumgebung: xfce

Accounts: s3bbl3r/test

</aside>

## Vorgehensweise

Beim erstmaligem Anmelden natürlich direkt Passwort-Probleme beim Eingeben in die Maske. Aber leider keine Hilfestellung dazu. Also nochmal komplett neu aufsetzen und penibel genau aufs PW achten. Deswegen wird jetzt zuerst ein Passwort ohne Sonderzeichen gesetzt, um die erste Anmeldung und konfigurieren des richtigen Tastatur-Layouts zu gewährleisten → **Erfolg!**

Konfigurieren der Shell-Farben von **fish** mit **fish_config -** Hintergrundfarbe muss über Fenstereinstellungen des Betriebssystems geregelt werden

- Installieren von tealdeer(tldr) über das Paketinstallationsprogramm in der grafischen Oberfläche und Erstellen des **tldr-cache** mit **tldr —update**
    
    ![image.png](Home%20Systeme%20Dokumentation/image%205.png)
    

Erstellen einer Datei “erste.datei” zu Testzwecken in **Dokumente/**  ausführbar machen mit **chmod +x**  und dem **Shebang !#/bin/fish** versehen (IHK-Konform für Bash mit **#!bin/bash**)

- ~~Hinzufügen  eines Themes für Bash mithilfe von **paru -S blesh**
dann verändern der Datei **~/.bashrc.**~~
    
    ```bash
    **# Am Anfang**
     [[ $- == *i* ]] && source /usr/share/blesh/ble.sh --noattach
    
    # AM Ende muss auch eine Ziele hinzugefügt werden
    
    -currently missing-
     
    ```
    
- Standard und erweitertes  Syntax-Highlighting hinzugefügt in ~/.nanorc
    
    ```bash
    # Standard-Syntaxhervorhebung von Nano (falls vorhanden)
    include "/usr/share/nano/*.nanorc"
    
    # Die erweiterten Schemata aus dem Paket 'nano-syntax-highlighting'
    include "/usr/share/nano-syntax-highlighting/*.nanorc"
    
    ```
    

## CachyOS for real

<aside>
💡

**Allgemeines**

Installieren der Developer Tools: **pacman -S base devel**

- Aliase
    
    Appletalk deaktivieren - alias net-pf-5 off 
    
</aside>

Standardinstallation 250 gb

TLDR installiert

Timeshift für Backups installiert zum Erstellen von Snapshots vor System Updates