# Projektidee: Automatisierte JupyterLab-Umgebung mit Ansible

In unserem Projekt möchten wir mit Ansible eine automatisierte JupyterLab-Umgebung aufbauen. Dabei wird eine Ubuntu-VM mit Vagrant erstellt und anschliessend automatisiert mit Ansible konfiguriert.

Auf der VM wird JupyterLab installiert und als eigener systemd-Service betrieben. Zusätzlich wird Nginx als lokaler Reverse Proxy eingesetzt. Dadurch kann neben der eigentlichen JupyterLab-Installation auch eine weitere Ansible-Rolle mit Template-Konfiguration umgesetzt werden.

Der Zugriff auf JupyterLab erfolgt bewusst nicht direkt über einen öffentlichen Port, sondern über einen SSH-Tunnel. Damit bleibt die Umgebung lokal, übersichtlich und sicher testbar.

Ziel des Projektes ist es, die komplette Installation und Grundkonfiguration einer lokalen JupyterLab-Umgebung reproduzierbar mit Ansible zu automatisieren.

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

## Was mit Ansible automatisiert wird

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

## Ansible-Bestandteile

Das Projekt verwendet die zentralen Ansible-Bausteine aus der Aufgabenstellung:

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

## Sicherheit

Die Umgebung ist bewusst lokal aufgebaut. JupyterLab wird nicht öffentlich freigegeben und läuft nicht als Root-Benutzer.

Private Keys, Tokens, Passwörter, `.env`-Dateien und lokale Vagrant-Dateien werden nicht ins Git-Repository hochgeladen.

Der JupyterLab-Token wird nur lokal verwendet und darf nicht im Repository gespeichert werden.

## Projektabgrenzung

Das Projekt ist keine produktive Plattform. Es verwendet kein Docker, kein Kubernetes, kein JupyterHub und keinen öffentlichen Webzugriff.

Der Fokus liegt auf einer schlanken, verständlichen und reproduzierbaren Ansible-Umsetzung für eine lokale Schulumgebung.
