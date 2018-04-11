**Installation**

Bevor es losgeht sollte man sich schon mal ein Letsencrypt-Zertifikat für die zu nutzende Domain besorgen und installieren. Wenn die Domain per https aufrufbar ist, geht die Installation auf dem Server im Terminal weiter. In dieser Anleitung benutze ich als Beispiel-Domain git.example.com.

Auf dem Server sollte mindestens git und supervisor installiert sein. Letzteres benutzen wir, um Gitea automatisiert zu starten. Gitea soll außerdem unter einem eigenen User laufen (git), der gleichzeitig auch für die SSH-Verbindungen genutzt werden kann.


    sudo apt-get install git supervisor
    sudo adduser --home /opt/git --shell /bin/bash --disabled-password git
    sudo mkdir -p /var/log/gitea && sudo chown git: /var/log/gitea
    sudo -u git -i

Das passende Binary von der Gitea Website herunterladen. Dort kann man auf der Downloadseite die aktuellste Version aufwählen (für stabilen Betrieb) oder master, wenn man gern gefährlich lebt, aber dafür die aktuellsten Features hat.


    cd /opt/git
    mkdir gitea
    cd gitea
    wget -O gitea https://dl.gitea.io/gitea/master/gitea-master-linux-amd64
    chmod +x gitea
    

Ob Gitea läuft kann man mit folgendem Aufruf testen:

    ./gitea web

An dieser Stelle sollte Gitea über http://git.example.com:3000 erreichbar sein.

**Plesk Subdomain inkl SSL Zertifikat von Let´s Encrypt:**
Als erstes solltet ihr in Plesk eine Subdomain anlegen, dabei gleich am besten ein Let´s Encrypt Zertifikat mit generieren lassen.
Danach zu Einstellungen für Apache & nginx gehen und bei Zusätzliche Anweisungen für HTTPS das hinzufügen:

    <Location />
    	ProxyPass http://127.0.0.1:3000/
    	ProxyPassReverse http://127.0.0.1:3000/
    </Location>

Zuletzt sorgen wir noch dafür, dass Gitea automatisch startet. Dazu reicht es eine Datei namens /etc/supervisor/conf.d/gitea.conf anzulegen mit folgendem Inhalt:


    [program:gitea]
    directory=/opt/git/gitea/
    command=/opt/git/gitea/gitea web
    autostart=true
    autorestart=true
    startsecs=10
    stdout_logfile=/var/log/gitea/stdout.log
    stdout_logfile_maxbytes=1MB
    stdout_logfile_backups=10
    stdout_capture_maxbytes=1MB
    stderr_logfile=/var/log/gitea/stderr.log
    stderr_logfile_maxbytes=1MB
    stderr_logfile_backups=10
    stderr_capture_maxbytes=1MB
    environment = HOME="/opt/git", USER="git"
    user = git

**Konfiguration**
Nach einem Neustart von Supervisor sollte Gitea nun laufen und die Seite müsste ohne Angabe des Ports über https://git.example.com erreichbar sein. Beim ersten Aufruf gelangt man auf die Installationsseite. Hier würde ich folgende Minimal-Konfiguration empfehlen:

    eine MySQL-Datenbank
    Repository: /opt/git/gitea/gitea-repositories
    Benutzer: git
    Domain: example.com (nicht git.example.com, es geht hier um die Domain für die SSH clone URLs)
    Anwendungs-URL: https://git.example.com
    Log-Pfad: /var/log/gitea
    E-Mail-Service: Ich empfehle einen externen Dienst wie Sparkpost oder Mailgun
    Diensteinstellungen: Nach Bedarf. Bei einem privaten Server würde ich die Registrierung deaktivieren und Seiten nur für angemeldete Benutzer zugänglich machen
    Admin: Hier kann man direkt das Admin-Konto anlegen


Gratulation. Jetzt sollte Gitea laufen. Viel Spaß beim nutzen!
Credits to: [Dakira](https://blog.yumdap.net/gitea-oder-gogs-statt-github-und-gitlab/) für das gute Tutorial