# Wazuh SIEM

> Wazuh on avatud lähtekoodiga turvaplatvorm, mis pakub ohutuvasust, logide analüüsi, failide terviklikkuse jälgimist ja haavatavuse skaneerimist.

<p align="center">
  <a href="21-nagios.md"><img src="https://img.shields.io/badge/Eelmine-Nagios-000000?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
</p>

---

## Eeldused

- Debian 12 server on seadistatud
- Vähemalt 4GB RAM (soovitatav 8GB)
- Vähemalt 50GB kettaruumi
- Kasutajal on sudo õigused
- Serveri hostname peab olema seadistatud

---

## Ettevalmistus

```bash
# Uuenda süsteemi
sudo apt update && sudo apt upgrade -y

# Paigalda vajalikud paketid (OLULINE!)
sudo apt install curl apt-transport-https gnupg2 software-properties-common lsb-release -y

# Kui eelmine käsk annab vea, proovi:
sudo apt --fix-broken install -y
sudo apt update
sudo apt install software-properties-common -y

# Kontrolli hostname (peab olema seadistatud!)
hostnamectl

# Ava vajalikud pordid (UFW puhul)
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 443/tcp
```

---

## Wazuh All-in-One paigaldamine

See paigaldab kõik komponendid ühele serverile: Wazuh manager, Wazuh indexer ja Wazuh dashboard.

### Meetod 1: Automaatne paigaldus (lihtne)

```bash
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.10/config.yml
```

Muuda `config.yml` faili (asenda IP-aadressid oma serveri IP-ga):
```bash
nano config.yml
```

```yaml
nodes:
  indexer:
    - name: wazuh-indexer
      ip: "10.0.80.10"
  server:
    - name: wazuh-server
      ip: "10.0.80.10"
  dashboard:
    - name: wazuh-dashboard
      ip: "10.0.80.10"
```

Käivita paigaldus:
```bash
sudo bash wazuh-install.sh -a -i
```

> **NB!** `-i` ignoreerib tervisekontrolli hoiatusi. Paigaldamine võib võtta 15-20 minutit.

### Meetod 2: Sammhaaval paigaldus

Kui automaatne ei tööta:

```bash
# 1. Genereeri konfiguratsioon
sudo bash wazuh-install.sh --generate-config-files

# 2. Paigalda Wazuh indexer
sudo bash wazuh-install.sh --wazuh-indexer wazuh-indexer

# 3. Käivita klaster
sudo bash wazuh-install.sh --start-cluster

# 4. Paigalda Wazuh server
sudo bash wazuh-install.sh --wazuh-server wazuh-server

# 5. Paigalda Dashboard
sudo bash wazuh-install.sh --wazuh-dashboard wazuh-dashboard
```

> **NB!** Jäta meelde paigalduse lõpus kuvatav admin parool!

---

## Teenuste kontrollimine

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

---

## Dashboard

Ava brauseris: `https://server-ip`

Sisselogimine:
- **Username:** admin
- **Password:** (paigalduse ajal genereeritud)

Parooli taastamine:
```bash
sudo tar -xvf wazuh-install-files.tar
cat wazuh-install-files/wazuh-passwords.txt
```

---

## Wazuh Agent paigaldamine

### Debian/Ubuntu klient

```bash
# GPG võtme lisamine
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg

# Repository lisamine
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# Agendi paigaldamine
sudo apt update
sudo apt install wazuh-agent -y
```

### Agendi konfigureerimine

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Muuda `<address>` väärtus:
```xml
<server>
  <address>wazuh-server-ip</address>
  <port>1514</port>
  <protocol>tcp</protocol>
</server>
```

### Agendi käivitamine

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

### AlmaLinux klient

```bash
sudo rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo cat > /etc/yum.repos.d/wazuh.repo << EOF
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF

sudo dnf install wazuh-agent -y
```

Konfigureeri ja käivita nagu Debianil.

---

### Windows klient

**1. Laadi alla agent:**

Ava PowerShell administraatorina:
```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi -OutFile wazuh-agent.msi
```

Või laadi alla brauseris: https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi

**2. Paigalda agent (GUI):**

- Käivita allalaaditud `.msi` fail
- Sisesta Wazuh Manager IP-aadress
- Sisesta agendi nimi (nt `DESKTOP-WIN10`)
- Lõpeta paigaldus

**3. Paigalda agent (käsureal):**

```powershell
msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER="10.0.80.10" WAZUH_AGENT_NAME="DESKTOP-WIN10" WAZUH_REGISTRATION_SERVER="10.0.80.10"
```

**4. Käivita teenus:**

```powershell
NET START WazuhSvc
```

**5. Kontrolli teenuse staatust:**

```powershell
Get-Service -Name WazuhSvc
```

**6. Agendi konfiguratsiooni muutmine:**

Konfiguratsiooni fail asub: `C:\Program Files (x86)\ossec-agent\ossec.conf`

```xml
<server>
  <address>WAZUH_MANAGER_IP</address>
  <port>1514</port>
  <protocol>tcp</protocol>
</server>
```

**7. Teenuse taaskäivitamine:**

```powershell
Restart-Service -Name WazuhSvc
```

**8. Logide asukoht:**

```
C:\Program Files (x86)\ossec-agent\ossec.log
```

---

## Tulemüür (Server)

```bash
sudo ufw allow 1514/tcp    # Agent ühendused
sudo ufw allow 1515/tcp    # Agent registreerimine
sudo ufw allow 55000/tcp   # Wazuh API
sudo ufw allow 443/tcp     # Dashboard (HTTPS)
```

---

## Kasulikud käsud

```bash
# Agentide nimekiri
sudo /var/ossec/bin/agent_control -l

# Agendi staatus
sudo /var/ossec/bin/agent_control -i <agent_id>

# Logide vaatamine
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Dashboard ei avane | Kontrolli `wazuh-dashboard` teenust ja port 443 |
| Agent ei ühendu | Kontrolli `<address>` ja pordid 1514/1515 |
| Indexer kollane | `journalctl -u wazuh-indexer` |
| Paigaldus ebaõnnestub | Kasuta `-i` lippu: `bash wazuh-install.sh -a -i` |
| Mälu viga | Suurenda swap: `sudo fallocate -l 4G /swapfile` |
| SSL viga | Kontrolli sertifikaate: `/etc/wazuh-indexer/certs/` |

---

## Paigalduse probleemide lahendamine

### Kui paigaldus katkeb

```bash
# Eemalda poolik paigaldus
sudo bash wazuh-install.sh --uninstall

# Puhasta
sudo apt remove --purge wazuh-* -y
sudo rm -rf /var/ossec /etc/wazuh-* /var/lib/wazuh-*
sudo rm -f wazuh-install-files.tar wazuh-install.sh config.yml

# Alusta uuesti
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
sudo bash wazuh-install.sh -a -i
```

### Indexer ei käivitu (mälu probleem)

```bash
# Lisa swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Lisa fstab-i püsivaks
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Taaskäivita indexer
sudo systemctl restart wazuh-indexer
```

### Logide kontrollimine

```bash
# Paigalduse logi
sudo cat /var/log/wazuh-install.log

# Indexer logi
sudo journalctl -u wazuh-indexer -f

# Manager logi
sudo tail -f /var/ossec/logs/ossec.log

# Dashboard logi
sudo journalctl -u wazuh-dashboard -f
```

---

## Levinud vead

- **Vale `<address>`** – agendi `ossec.conf` failis peab olema serveri IP
- **Tulemüür** – pordid 1514/TCP ja 1515/TCP peavad olema avatud
- **Vähe mälu** – Wazuh indexer vajab vähemalt 2GB vaba RAM-i
- **Hostname puudu** – sea hostname enne paigaldust: `sudo hostnamectl set-hostname wazuh-server`
- **config.yml vale** – kontrolli, et IP-aadressid on õiged
