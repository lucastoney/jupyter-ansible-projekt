# Projektidee: Meine automatisierte JupyterLab-Umgebung mit Ansible

In meinem Projekt möchte ich mit Ansible eine automatisierte JupyterLab-Umgebung aufbauen. Dabei erstelle ich eine Ubuntu-VM mit Vagrant und konfiguriere sie anschliessend automatisiert mit Ansible.

Auf der VM installiere ich JupyterLab und betreibe es als eigenen systemd-Service. Zusätzlich setze ich Nginx als lokalen Reverse Proxy ein. Dadurch kann ich neben der eigentlichen JupyterLab-Installation auch eine weitere Ansible-Rolle mit Template-Konfiguration umsetzen.

Der Zugriff auf JupyterLab erfolgt bewusst nicht direkt über einen öffentlichen Port, sondern über einen SSH-Tunnel. Damit bleibt die Umgebung lokal, übersichtlich und sicher testbar.

Mein Ziel ist es, die komplette Installation und Grundkonfiguration einer lokalen JupyterLab-Umgebung reproduzierbar mit Ansible zu automatisieren.

## Architektur grosso modo

Die Umgebung besteht aus einem Laptop/PC und einer Linux-VM in einem privaten Netzwerk.

### Laptop / PC

* Vagrant
* VirtualBox
* Ansible Controller
* Git-Arbeitsumgebung
* Browser

### Ubuntu-VM `jupyter-node`

* Managed Host
* JupyterLab
* Nginx Reverse Proxy
* UFW Firewall
* eigener Linux-Benutzer für JupyterLab

## Netzwerk / Zugriff

Die VM befindet sich in einem privaten Host-only-Netzwerk und verwendet die IP-Adresse:

```text
192.168.56.20
```

Ansible verbindet sich vom Laptop/PC per SSH mit der VM und führt dort die Konfiguration aus.

JupyterLab und Nginx werden nur lokal innerhalb der VM gebunden:

```text
JupyterLab: 127.0.0.1:8888
Nginx:      127.0.0.1:8080
```

Der Zugriff vom Browser erfolgt über einen SSH-Tunnel.

```text
Benutzer
  |
  | Webbrowser
  |
  | SSH-Tunnel
  v
Ubuntu-VM jupyter-node
  |
  | Nginx Reverse Proxy
  v
JupyterLab
```

## Was ich mit Ansible automatisiere

* Grundinstallation und Systemvorbereitung der VM
* Installation benötigter Pakete
* Erstellung eines eigenen JupyterLab-Benutzers
* Einrichtung der Verzeichnisstruktur
* Erstellung eines Python Virtual Environments
* Installation von JupyterLab
* Erstellung eines systemd-Services für JupyterLab
* Installation und Konfiguration von Nginx
* Einrichtung von Nginx als lokaler Reverse Proxy
* Konfiguration der Firewall mit UFW
* Starten und Aktivieren der benötigten Services
* Bereitstellung eines Beispiel-Notebooks

## Verwendete Ansible-Bestandteile

Mein Projekt verwendet die zentralen Ansible-Bausteine aus der Aufgabenstellung:

* Inventory
* Variablen
* Playbook
* Rollen
* Templates
* Dateien

Die Rollen sind einfach und klar getrennt:

```text
base
jupyterlab
nginx_proxy
```

## Sicherheitsansatz

Die Umgebung ist bewusst lokal aufgebaut. JupyterLab wird nicht öffentlich freigegeben und läuft nicht als Root-Benutzer.

Private Keys, Tokens, Passwörter, `.env`-Dateien und lokale Vagrant-Dateien werden nicht ins Git-Repository hochgeladen.

Der JupyterLab-Token wird nur lokal verwendet und darf nicht im Repository gespeichert werden.

## Projektabgrenzung

Mein Projekt ist keine produktive Plattform. Es verwendet kein Docker, kein Kubernetes, kein JupyterHub und keinen öffentlichen Webzugriff.

Der Fokus liegt auf einer schlanken, verständlichen und reproduzierbaren Ansible-Umsetzung für eine lokale Schulumgebung.
