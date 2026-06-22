# Projektarbeit: Automatisierte Bereitstellung eines JupyterLab-Servers mit Ansible

## 1. Titelblatt / Projektinformationen

**Projekt:** Automatisierte Bereitstellung eines lokal abgesicherten JupyterLab-Servers mit Ansible und Nginx  
**Modul:** Netzwerkbetriebssysteme / Automatisation mit Ansible  
**Arbeitsform:** Einzelarbeit  
**System:** Ubuntu 24.04 LTS in VirtualBox, gesteuert mit Vagrant  
**Controller:** Laptop/PC mit Ansible  
**Managed Host:** VM `jupyter-node` mit IP `192.168.56.20`

## 2. Ausgangslage

Die Aufgabenstellung verlangt ein schlankes Automatisierungsprojekt mit Ansible. Das Projekt soll ein Inventory, Variablen, ein Playbook, Rollen und Templates enthalten. Der Ansible Controller soll sich per SSH mit einem Managed Host verbinden und diesen automatisiert konfigurieren.

## 3. Zielsetzung

Ziel ist eine reproduzierbare VM, auf der JupyterLab automatisiert installiert und lokal abgesichert betrieben wird. Zusaetzlich wird Nginx als lokaler Reverse Proxy eingerichtet, damit eine weitere Ansible-Rolle mit Template demonstriert werden kann.

## 4. Architektur

Der Laptop/PC uebernimmt mehrere Aufgaben: VirtualBox-Host, Vagrant-Steuerung, Ansible Controller, Git-Arbeitsumgebung und Browser-Client. Die Ubuntu-VM `jupyter-node` ist der Managed Host.

Die Verbindung laeuft per SSH:

```text
Laptop / PC
Ansible Controller
        |
        | SSH Port 22
        v
Ubuntu-VM jupyter-node
├── JupyterLab: 127.0.0.1:8888
└── Nginx:      127.0.0.1:8080 -> 127.0.0.1:8888
```

Der Browser greift nicht direkt auf die VM-Ports zu. Stattdessen wird ein SSH-Tunnel vom Laptop zur VM aufgebaut.

## 5. Sicherheitskonzept

JupyterLab laeuft nicht als Root, sondern als eigener Benutzer `jupyter`. Der Dienst bindet nur auf `127.0.0.1:8888`. Nginx bindet nur auf `127.0.0.1:8080`. Die Ports `8888` und `8080` werden nicht in VirtualBox weitergeleitet.

Private SSH-Keys, Tokens, Passwoerter, `.env`-Dateien und `vagrant/.vagrant/`-Dateien werden durch `.gitignore` vom Repository ausgeschlossen. Jupyter-Tokens werden nur lokal zur Laufzeit aus dem Journal gelesen.

## 6. Technische Umsetzung mit Ansible

Das Playbook `ansible/site.yml` fuehrt drei Rollen aus:

- `base`: installiert die benoetigten Basispakete.
- `jupyterlab`: erstellt Benutzer und Verzeichnisse, installiert JupyterLab in einem Python Virtual Environment und richtet einen systemd-Service ein.
- `nginx_proxy`: erstellt eine lokale Nginx-Site aus einem Template und leitet Anfragen an JupyterLab weiter.

Zentrale Einstellungen liegen in `ansible/group_vars/all.yml`. Dazu gehoeren Paketliste, Benutzername, Pfade, lokale IPs und Ports. Die systemd- und Nginx-Konfigurationen werden ueber Jinja2-Templates erzeugt.

## 7. Test und Funktionsnachweis

Ansible Ping:

```bash
ansible -i ansible/inventory.ini all -m ping
```

Erwartung: `pong`

Playbook ausfuehren:

```bash
ansible-playbook -i ansible/inventory.ini ansible/site.yml
```

Erwartung: `failed=0`

JupyterLab-Service:

```bash
ssh vagrant@192.168.56.20 "sudo systemctl status jupyterlab --no-pager"
```

Erwartung: `active (running)`

Nginx-Service:

```bash
ssh vagrant@192.168.56.20 "sudo systemctl status nginx --no-pager"
```

Erwartung: `active (running)`

Portbindung:

```bash
ssh vagrant@192.168.56.20 "ss -tulpen | grep -E '8888|8080'"
```

Erwartung: `127.0.0.1:8888` und `127.0.0.1:8080`, nicht `0.0.0.0`.

## 8. Demo-Ablauf fuer den Dozenten

1. Repository zeigen und Struktur erklaeren.
2. In den Ordner `vagrant/` wechseln und `vagrant up` ausfuehren oder gestartete VM zeigen.
3. SSH-Verbindung aus dem Ordner `vagrant/` mit `vagrant ssh` testen.
4. Ansible Ping ausfuehren.
5. Playbook ausfuehren.
6. Services und Portbindungen pruefen.
7. SSH-Tunnel starten.
8. JupyterLab im Browser oeffnen.
9. Beispiel-Notebook ausfuehren.
10. Sicherheitsaspekte und `.gitignore` zeigen.

## 9. Probleme und Loesungen

Ein moegliches Problem ist ein fehlender SSH-Key, wenn die VM noch nicht mit `vagrant up` im Ordner `vagrant/` erstellt wurde. Die Loesung ist, zuerst Vagrant auszufuehren und danach Ansible zu starten.

Wenn JupyterLab im Browser nicht erreichbar ist, muss zuerst geprueft werden, ob der SSH-Tunnel noch aktiv ist und ob der Dienst auf `127.0.0.1:8888` laeuft.

## 10. Fazit

Das Projekt zeigt eine klassische Ansible-Architektur mit Controller und Managed Host. Die Installation ist reproduzierbar, die Konfiguration ist in Rollen strukturiert und die sicherheitsrelevanten Ports bleiben lokal gebunden. Nginx erweitert das Projekt sinnvoll, ohne es unnoetig gross zu machen.

## 11. Moegliche Erweiterungen

Moegliche Erweiterungen waeren eine genauere Versionspinning-Strategie fuer Python-Pakete, zusaetzliche Tests mit Molecule oder eine HTTPS-Konfiguration fuer ein reales Netzwerk. Fuer dieses Schulprojekt bleiben diese Punkte bewusst ausserhalb des Umfangs.

## 12. Abgrenzung

Nicht umgesetzt werden Docker, Kubernetes, JupyterHub, Internet-Zugriff, oeffentliche Jupyter-Ports, oeffentliche Nginx-Ports, komplexe Benutzerverwaltung oder produktive Datenplattform-Funktionen. Das Projekt bleibt absichtlich schlank und fokussiert auf Ansible.
