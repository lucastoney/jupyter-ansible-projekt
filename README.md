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
