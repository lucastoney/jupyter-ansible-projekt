# Projektansage: Automatisierte Bereitstellung einer JupyterLab-Umgebung mit Ansible

Ich erstelle ein schlankes Automatisierungsprojekt mit Ansible. Eine Ubuntu-VM wird mit Vagrant und VirtualBox bereitgestellt und anschliessend vom Laptop/PC als Ansible Controller über SSH konfiguriert. Auf der VM wird JupyterLab automatisiert installiert und als systemd-Service betrieben. Ziel ist eine einfache, reproduzierbare und nachvollziehbare Umgebung, welche die zentralen Ansible-Bausteine der Aufgabenstellung zeigt.

## Eingesetzte Technologien

### Mindestumfang

* Vagrant
* VirtualBox
* Ubuntu 24.04 LTS
* Ansible
* JupyterLab
* systemd
* SSH Public-Key Zugriff

### Optionale Erweiterung / Zusatznutzen je nach Zeit

* Nginx als lokaler Reverse Proxy
* UFW als einfache Firewall-Grundkonfiguration
* SSH-Tunnel für den lokalen Browserzugriff

## Grobe Architektur

Der Laptop/PC dient als VirtualBox-Host, Vagrant-Steuerung, Ansible Controller, Git-Arbeitsumgebung und Browser-Client. Die Ubuntu-VM `jupyter-node` ist der Managed Host. Ansible verbindet sich per SSH zur VM und führt dort die Rollen aus.

Im Mindestumfang werden die Rollen `base` und `jupyterlab` umgesetzt. Als sinnvolle Erweiterung kann zusätzlich die Rolle `nginx_proxy` ergänzt werden.

## Sicherheitsansatz

JupyterLab läuft nicht als Root, sondern unter einem eigenen Linux-Benutzer. Der Dienst wird lokal gebunden und nicht bewusst als öffentlich erreichbarer Dienst aufgebaut. Private Keys, Tokens, Passwörter, `.env`-Dateien und lokale Vagrant-Dateien werden nicht ins Git-Repository aufgenommen.

Falls die optionale Nginx-Erweiterung umgesetzt wird, lauscht Nginx ebenfalls nur lokal und leitet intern an JupyterLab weiter. Der Zugriff erfolgt dann über einen lokalen SSH-Tunnel.

## Resultat

Am Ende entsteht ein reproduzierbares und gut dokumentiertes Ansible-Projekt mit Inventory, Variablen, Playbook, Rollen, Templates, systemd-Service, Architekturzeichnung und nachvollziehbaren Testbefehlen.

Der verbindliche Massstab ist die funktionierende automatisierte Bereitstellung von JupyterLab mit Ansible.

