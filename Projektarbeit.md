# Projektarbeit

## Automatisierte JupyterLab-Umgebung mit Ansible

**TEKO Schweizerische Fachschule Bern**  
**Modul:** Netzwerkbetriebssysteme / Automatisation mit Ansible  
**Arbeitsform:** Einzelarbeit  
**Projekt:** JupyterLab mit Ansible, Vagrant und lokalem Nginx-Reverse-Proxy  
**Managed Host:** Ubuntu-VM `jupyter-node` mit IP `192.168.56.20`

---

## 1. Ausgangslage

Für dieses Projekt soll eine kleine, reproduzierbare Umgebung mit Ansible automatisiert bereitgestellt werden. Die Aufgabenstellung verlangt zentrale Ansible-Bestandteile wie Inventory, Variablen, Playbook, Rollen und Templates.

Die Umgebung besteht aus einem Laptop/PC als Ansible Controller und einer Ubuntu-VM als Managed Host. Die VM wird mit Vagrant erstellt und danach über SSH mit Ansible konfiguriert.

## 2. Ziel

Ziel ist eine lokale JupyterLab-Umgebung, die mit wenigen Schritten reproduziert werden kann. JupyterLab wird automatisiert installiert und als systemd-Service betrieben. Zusätzlich wird Nginx als lokaler Reverse Proxy eingerichtet, um eine weitere Ansible-Rolle mit Template-Konfiguration zu zeigen.

Der Zugriff auf JupyterLab erfolgt nicht über einen öffentlich freigegebenen Port, sondern über einen SSH-Tunnel.

## 3. Architektur

```text
Laptop / PC
Ansible Controller + Browser
        |
        | SSH Port 22
        v
Ubuntu-VM jupyter-node
├── JupyterLab: 127.0.0.1:8888
└── Nginx:      127.0.0.1:8080 -> 127.0.0.1:8888
```

Der Laptop/PC dient als VirtualBox-Host, Vagrant-Steuerung, Ansible Controller, Git-Arbeitsumgebung und Browser-Client. Die VM `jupyter-node` ist der Managed Host.

## 4. Sicherheitskonzept

JupyterLab läuft nicht als Root, sondern unter dem eigenen Benutzer `jupyter`. JupyterLab bindet nur auf `127.0.0.1:8888`. Nginx bindet nur auf `127.0.0.1:8080` und leitet lokal an JupyterLab weiter.

Die Ports `8888` und `8080` werden nicht in VirtualBox weitergeleitet. Der Zugriff erfolgt über SSH-Tunnel. Private Keys, Tokens, Passwörter, `.env`-Dateien und lokale Vagrant-Dateien werden nicht ins Git-Repository aufgenommen.

## 5. Technische Umsetzung

Das Projekt ist in Vagrant und Ansible aufgeteilt:

- `vagrant/Vagrantfile`: erstellt die Ubuntu-VM
- `ansible/inventory.ini`: definiert den Managed Host
- `ansible/group_vars/all.yml`: enthält zentrale Variablen
- `ansible/site.yml`: führt die Rollen aus
- `roles/base`: installiert Basispakete
- `roles/jupyterlab`: installiert und betreibt JupyterLab
- `roles/nginx_proxy`: konfiguriert Nginx als lokalen Reverse Proxy

Die systemd- und Nginx-Konfigurationen werden über Templates erzeugt.

## 6. Reproduktion der Umgebung

VM starten:

```bash
cd vagrant
vagrant up
cd ..
```

SSH-Verbindung prüfen:

```bash
cd vagrant
vagrant ssh
exit
cd ..
```

Ansible-Verbindung prüfen:

```bash
ansible -i ansible/inventory.ini all -m ping
```

Erwartung:

```text
pong
```

Playbook ausführen:

```bash
ansible-playbook -i ansible/inventory.ini ansible/site.yml
```

Erwartung:

```text
failed=0
```

## 7. Funktionsnachweis

JupyterLab-Service prüfen:

```bash
ssh vagrant@192.168.56.20 "sudo systemctl status jupyterlab --no-pager"
```

Nginx-Service prüfen:

```bash
ssh vagrant@192.168.56.20 "sudo systemctl status nginx --no-pager"
```

Portbindung prüfen:

```bash
ssh vagrant@192.168.56.20 "ss -tulpen | grep -E '8888|8080'"
```

Erwartung:

```text
127.0.0.1:8888
127.0.0.1:8080
```

Nicht erwartet:

```text
0.0.0.0:8888
0.0.0.0:8080
```

SSH-Tunnel direkt zu JupyterLab:

```bash
ssh -L 8888:localhost:8888 vagrant@192.168.56.20
```

Browser:

```text
http://localhost:8888
```

SSH-Tunnel über Nginx:

```bash
ssh -L 8080:localhost:8080 vagrant@192.168.56.20
```

Browser:

```text
http://localhost:8080
```

## 8. Demo-Ablauf

1. Repository und Architekturzeichnung zeigen.
2. VM mit `vagrant up` starten.
3. SSH-Verbindung mit `vagrant ssh` prüfen.
4. Ansible Ping ausführen.
5. Playbook ausführen.
6. Services und lokale Portbindung prüfen.
7. SSH-Tunnel starten.
8. JupyterLab im Browser öffnen.
9. Beispiel-Notebook ausführen.

## 9. Probleme und Lösungen

Wenn Ansible keinen SSH-Zugriff erhält, wurde die VM möglicherweise noch nicht mit `vagrant up` erstellt. In diesem Fall muss zuerst die VM im Ordner `vagrant/` gestartet werden.

Wenn JupyterLab im Browser nicht erreichbar ist, muss geprüft werden, ob der SSH-Tunnel noch aktiv ist und ob der Dienst auf `127.0.0.1:8888` läuft.

## 10. Fazit

Das Projekt zeigt eine klassische Ansible-Architektur mit Controller und Managed Host. Die Umgebung ist bewusst schlank gehalten und kann mit Vagrant und Ansible reproduziert werden. JupyterLab und Nginx bleiben lokal gebunden, wodurch der Zugriff kontrolliert über SSH-Tunnel erfolgt.

## 11. Abgrenzung

Nicht umgesetzt werden Docker, Kubernetes, JupyterHub, öffentlicher Webzugriff, komplexe Benutzerverwaltung oder eine produktive Datenplattform. Der Fokus liegt auf einer verständlichen Ansible-Umsetzung für eine lokale Schulumgebung.
