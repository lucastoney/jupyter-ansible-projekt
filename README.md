# Automatisierte JupyterLab-Umgebung mit Ansible

In diesem Projekt wird eine Ubuntu-VM mit Vagrant erstellt und anschliessend mit Ansible automatisiert konfiguriert. Auf der VM wird JupyterLab installiert und als systemd-Service betrieben.

Zusätzlich wird Nginx als lokaler Reverse Proxy eingerichtet. JupyterLab und Nginx sind nicht öffentlich erreichbar, sondern nur lokal auf der VM gebunden. Der Zugriff erfolgt über einen SSH-Tunnel.

Ziel des Projektes ist eine schlanke, reproduzierbare und sichere JupyterLab-Umgebung, welche die zentralen Ansible-Bausteine der Aufgabenstellung zeigt.

## Architektur

Die Umgebung besteht aus einem Laptop/PC und einer Ubuntu-VM in einem privaten Netzwerk.

**Laptop / PC**

- VirtualBox-Host
- Vagrant-Steuerung
- Ansible Controller
- Browser-Client

**Ubuntu-VM `jupyter-node`**

- Managed Host
- JupyterLab
- Nginx Reverse Proxy
- UFW Firewall

## Netzwerk / Zugriff

```text
Laptop / PC
Ansible Controller + Browser
        |
        | SSH
        v
Ubuntu-VM jupyter-node
├── JupyterLab: 127.0.0.1:8888
└── Nginx:      127.0.0.1:8080 -> JupyterLab
```

JupyterLab und Nginx werden nicht über öffentliche Ports freigegeben. Der Zugriff erfolgt lokal über SSH-Tunnel.

## Was mit Ansible automatisiert wird

- Systemvorbereitung der VM
- Installation benötigter Pakete
- Erstellung eines eigenen JupyterLab-Benutzers
- Installation von JupyterLab
- Einrichtung eines systemd-Services
- Konfiguration von Nginx als lokalem Reverse Proxy
- Grundkonfiguration der Firewall
- Starten und Aktivieren der Services

## Verwendete Ansible-Bestandteile

- Inventory
- Variablen
- Playbook
- Rollen
- Templates
- Dateien

Die Rollen sind:

```text
base
jupyterlab
nginx_proxy
```

## Sicherheit

JupyterLab läuft nicht als Root. Die Dienste binden nur auf `127.0.0.1`. Private Keys, Tokens, Passwörter, `.env`-Dateien und lokale Vagrant-Dateien werden nicht ins Repository aufgenommen.

## Abgrenzung

Das Projekt verwendet kein Docker, kein Kubernetes, kein JupyterHub und keinen öffentlichen Webzugriff. Der Fokus liegt auf einer einfachen und nachvollziehbaren Ansible-Umsetzung.
