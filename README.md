# Bruteforcelab - HackMyVM (Easy)

![Bruteforcelab.png](Bruteforcelab.png)

## Übersicht

*   **VM:** Bruteforcelab
*   **Plattform:** [https://hackmyvm.eu/machines/machine.php?vm=Bruteforcelab](https://hackmyvm.eu/machines/machine.php?vm=Bruteforcelab)
*   **Schwierigkeit:** Easy
*   **Autor der VM:** DarkSpirit
*   **Datum des Writeups:** 10. April 2023
*   **Original-Writeup:** [https://alientec1908.github.io/Bruteforcelab_HackMyVM_Easy/](https://alientec1908.github.io/Bruteforcelab_HackMyVM_Easy/)
*   **Autor:** Ben C.

## Kurzbeschreibung

Die Challenge "Bruteforcelab" auf HackMyVM (Schwierigkeit: Easy) fokussiert sich, wie der Name schon andeutet, auf das Thema Brute-Force. Ziel ist es, durch systematisches Ausprobieren von Anmeldeinformationen zunächst einen initialen Benutzerzugriff via SSH zu erlangen und anschließend durch einen weiteren, gezielten Brute-Force-Angriff auf das Root-Passwort volle Systemkontrolle zu erreichen. Die Maschine dient als gute Übung für den Umgang mit Brute-Force-Tools und das Erkennen von Hinweisen auf schwache Passwörter.

## Disclaimer / Wichtiger Hinweis

Die in diesem Writeup beschriebenen Techniken und Werkzeuge dienen ausschließlich zu Bildungszwecken im Rahmen von legalen Capture-The-Flag (CTF)-Wettbewerben und Penetrationstests auf Systemen, für die eine ausdrückliche Genehmigung vorliegt. Die Anwendung dieser Methoden auf Systeme ohne Erlaubnis ist illegal. Der Autor übernimmt keine Verantwortung für missbräuchliche Verwendung der hier geteilten Informationen. Handeln Sie stets ethisch und verantwortungsbewusst.

## Verwendete Tools

*   `arp-scan`
*   `nmap`
*   `nikto`
*   `hydra`
*   `ssh`
*   `find`
*   `wget`
*   `bash` (für ein Brute-Force-Skript via `su`)
*   Standard Linux-Befehle (`ls`, `cat`, `echo`, `chmod`, `sudo`, `id`, etc.)

## Lösungsweg (Zusammenfassung)

Der Angriff auf die Maschine "Bruteforcelab" gliederte sich in folgende Phasen:

1.  **Reconnaissance & Enumeration:**
    *   IP-Findung mittels `arp-scan` (`192.168.2.122`).
    *   `nmap`-Scan identifizierte offene Ports: SSH (22/tcp - OpenSSH 8.4p1), Webmin (10000/tcp - MiniServ 2.021), und zwei Samba-Dienste (19000/tcp, 19222/tcp - Samba smbd 4.6.2).

2.  **Web Enumeration (Port 10000):**
    *   Manuelle Tests auf SQL-Injection auf der Webmin-Login-Seite (`/session_login.cgi`) blieben erfolglos; die Anwendung blockierte Sonderzeichen.
    *   `nikto` fand fehlende Sicherheitsheader, einen Zertifikats-Mismatch und bestätigte MiniServ/Webmin. Es wurden viele generische CGI-Pfade getestet, die aber nicht direkt relevant waren.

3.  **Initial Access (SSH Brute-Force):**
    *   Ein Brute-Force-Angriff mit `hydra` auf den SSH-Dienst (Port 22) für den vermuteten Benutzernamen `andrea` und die Passwortliste `rockyou.txt` war erfolgreich.
    *   Das Passwort `awesome` für den Benutzer `andrea` wurde gefunden.
    *   Erfolgreicher SSH-Login als `andrea` mit den gefundenen Credentials.

4.  **Post-Exploitation / User-Flag:**
    *   Als Benutzer `andrea` wurde `sudo -l` ausgeführt, was zeigte, dass `andrea` keine `sudo`-Rechte hat.
    *   Die User-Flag wurde in `/home/andrea/user.txt` gefunden.
    *   Die Suche nach SUID-Binaries mit `find` zeigte Standard-Linux-Binaries, aber keine offensichtlich ausnutzbaren benutzerdefinierten SUIDs.

5.  **Privilege Escalation (von `andrea` zu `root` durch `su` Brute-Force):**
    *   Basierend auf dem Namen der Maschine ("Bruteforcelab") und der Tatsache, dass `su` SUID root ist, wurde ein gezielter Brute-Force-Angriff auf das Root-Passwort vermutet.
    *   Ein Bash-Skript (`crack_su_password_num.sh`) wurde erstellt, das Jahreszahlen (1990-2000) als Root-Passwort via `su root -c whoami` testet.
    *   Das Skript wurde auf das Zielsystem (`/tmp`) hochgeladen, ausführbar gemacht und ausgeführt.
    *   Das Skript identifizierte `1998` als das Passwort für den `root`-Benutzer.
    *   Erfolgreicher Wechsel zum `root`-Benutzer mit `su root` und dem Passwort `1998`.

## Wichtige Schwachstellen und Konzepte

*   **Schwaches SSH-Passwort:** Der Benutzer `andrea` verwendete ein Passwort (`awesome`), das in gängigen Wortlisten enthalten ist und per Brute-Force (mit `hydra`) geknackt werden konnte.
*   **Schwaches Root-Passwort:** Das Root-Passwort (`1998`) war extrem schwach (eine einfache Jahreszahl) und konnte durch ein gezieltes Brute-Force-Skript über `su` erraten werden.
*   **Brute-Force-Angriffe:** Die zentrale Methode zum Erlangen des initialen Zugriffs und der Root-Rechte.
*   **SUID-Binary `su`:** Obwohl `su` selbst keine Schwachstelle ist, ermöglichte seine SUID-Root-Eigenschaft in Kombination mit einem schwachen Root-Passwort die Privilegieneskalation.
*   **Informationslecks durch Webmin/Nikto:** Obwohl nicht direkt ausgenutzt, lieferte die Enumeration des Webmin-Dienstes Informationen über den Server und fehlende Sicherheitskonfigurationen.

## Flags

*   **User Flag (`/home/andrea/user.txt`):** `d5eb7d8b6f57c295e0bedf7eef531360`
*   **Root Flag (`/root/root.txt`):** `d5eb7d8b6f57c295e0bedf7eef531360` 
    *(Hinweis: Die Flags waren im Writeup identisch weil der Part nicht mehr durchgearbeitet wurde)*

## Tags

`HackMyVM`, `Bruteforcelab`, `Easy`, `Brute Force`, `SSH`, `SUID`, `Password Cracking`, `Linux`, `Hydra`, `Bash Scripting`
