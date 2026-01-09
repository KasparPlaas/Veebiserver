# Wazuh SIEM

> Wazuh on avatud lähtekoodiga turvaplatvorm, mis pakub ohutuvasust, logide analüüsi, failide terviklikkuse jälgimist ja haavatavuse skaneerimist.

<p align="center">
  <a href="21-nagios.md"><img src="https://img.shields.io/badge/Eelmine-Nagios-000000?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
</p>

---

## Eeldused

| Nõue | Miinimum | Soovitatav |
|------|----------|------------|
| RAM | 4 GB | 8 GB |
| Kettaruum | 50 GB | 100 GB |
| CPU | 2 tuuma | 4 tuuma |
| OS | Debian 12 / Ubuntu 22.04 | Debian 12 |

---

# 1. ETAPP: Süsteemi ettevalmistus

> **OLULINE!** Ära jäta ühtegi sammu vahele!

## 1.1 Süsteemi uuendamine

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt autoremove -y
```

## 1.2 Vajalike pakettide paigaldamine

```bash
sudo apt install -y \
    curl \
    wget \
    gnupg2 \
    apt-transport-https \
    software-properties-common \
    lsb-release \
    ca-certificates \
    debconf \
    adduser \
    procps
```

**Kui tekib viga `software-properties-common` paketiga:**

```bash
# Paranda katkised paketid
sudo apt --fix-broken install -y
sudo dpkg --configure -a

# Puhasta cache
sudo apt clean
sudo apt update

# Proovi uuesti
sudo apt install software-properties-common -y
```

## 1.3 Hostname seadistamine

```bash
# Kontrolli praegust
hostname

# Sea uus hostname
sudo hostnamectl set-hostname wazuh-server

# Lisa /etc/hosts faili
echo "127.0.1.1 wazuh-server" | sudo tee -a /etc/hosts

# Kontrolli
hostnamectl
```

## 1.4 Swap lisamine (kui RAM < 8GB)

```bash
# Kontrolli olemasolevat swap
free -h

# Loo 4GB swap fail
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Tee püsivaks
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Kontrolli
free -h
```

## 1.5 Tulemüüri seadistamine

```bash
# Ava vajalikud pordid ENNE paigaldust
sudo ufw allow 443/tcp comment 'Wazuh Dashboard'
sudo ufw allow 1514/tcp comment 'Wazuh Agent'
sudo ufw allow 1515/tcp comment 'Wazuh Registration'
sudo ufw allow 55000/tcp comment 'Wazuh API'
sudo ufw allow 9200/tcp comment 'Wazuh Indexer'

# Kontrolli
sudo ufw status
```

---

# 2. ETAPP: Wazuh paigaldamine

## 2.1 Skriptide allalaadimine

```bash
cd /tmp

# Laadi alla paigaldusskript
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh

# Laadi alla konfiguratsioonifail
curl -sO https://packages.wazuh.com/4.10/config.yml
```

## 2.2 Konfiguratsioonifaili muutmine

```bash
nano config.yml
```

**Asenda IP-aadressid oma serveri IP-ga:**

```yaml
nodes:
  # Wazuh indexer nodes
  indexer:
    - name: node-1
      ip: "SINU_SERVERI_IP"

  # Wazuh server nodes
  server:
    - name: wazuh-1
      ip: "SINU_SERVERI_IP"

  # Wazuh dashboard nodes
  dashboard:
    - name: dashboard
      ip: "SINU_SERVERI_IP"
```

**Näide (IP: 10.0.80.10):**
```yaml
nodes:
  indexer:
    - name: node-1
      ip: "10.0.80.10"
  server:
    - name: wazuh-1
      ip: "10.0.80.10"
  dashboard:
    - name: dashboard
      ip: "10.0.80.10"
```

## 2.3 Paigalduse käivitamine

```bash
sudo bash wazuh-install.sh -a -i
```

| Lipp | Kirjeldus |
|------|-----------|
| `-a` | All-in-one (kõik komponendid) |
| `-i` | Ignoreeri tervisekontrolli hoiatusi |

> **Oota 15-30 minutit.** Ära katkesta protsessi!

## 2.4 Paigalduse lõpp

Paigalduse lõpus kuvatakse:
```
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
    User: admin
    Password: <RANDOM_PASSWORD>
```

**SALVESTA SEE PAROOL!**

---

# 3. ETAPP: Kontrollimine

## 3.1 Teenuste staatus

```bash
# Kõik teenused
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard

# Eraldi
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
```

Kõik peavad olema `active (running)`.

## 3.2 Portide kontrollimine

```bash
sudo ss -tlnp | grep -E '443|1514|1515|9200|55000'
```

## 3.3 Dashboard

Ava brauseris: `https://SINU_SERVERI_IP`

- **Username:** admin
- **Password:** (paigalduse ajal kuvatud)

## 3.4 Parooli taastamine

```bash
sudo tar -xvf /tmp/wazuh-install-files.tar -C /tmp
cat /tmp/wazuh-install-files/wazuh-passwords.txt
```

---

# 4. Wazuh Agent paigaldamine

## 4.1 Debian/Ubuntu agent

```bash
# 1. GPG võti
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && sudo chmod 644 /usr/share/keyrings/wazuh.gpg

# 2. Repository
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list

# 3. Paigalda
sudo apt update
sudo apt install wazuh-agent -y

# 4. Konfigureeri (MUUDA IP!)
sudo sed -i 's/MANAGER_IP/10.0.80.10/' /var/ossec/etc/ossec.conf

# 5. Käivita
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent

# 6. Kontrolli
sudo systemctl status wazuh-agent
```

**Käsitsi konfiguratsioon:**
```bash
sudo nano /var/ossec/etc/ossec.conf
```

Leia ja muuda:
```xml
<client>
  <server>
    <address>10.0.80.10</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

## 4.2 AlmaLinux/Rocky agent

```bash
# 1. GPG võti
sudo rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH

# 2. Repository
cat << EOF | sudo tee /etc/yum.repos.d/wazuh.repo
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF

# 3. Paigalda
sudo dnf install wazuh-agent -y

# 4. Konfigureeri
sudo sed -i 's/MANAGER_IP/10.0.80.10/' /var/ossec/etc/ossec.conf

# 5. Käivita
sudo systemctl daemon-reload
sudo systemctl enable --now wazuh-agent
```

## 4.3 Windows agent

### Allalaadimine

PowerShell (administraator):
```powershell
# Laadi alla
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.10.0-1.msi" -OutFile "$env:TEMP\wazuh-agent.msi"

# Paigalda (MUUDA IP!)
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q WAZUH_MANAGER="10.0.80.10" WAZUH_AGENT_NAME="$env:COMPUTERNAME" WAZUH_REGISTRATION_SERVER="10.0.80.10"

# Käivita teenus
NET START WazuhSvc

# Kontrolli
Get-Service WazuhSvc
```

### GUI paigaldus

1. Laadi alla: https://packages.wazuh.com/4.x/windows/wazuh-agent-4.10.0-1.msi
2. Käivita `.msi` fail
3. Sisesta **Manager IP**: `10.0.80.10`
4. Sisesta **Agent nimi**: `DESKTOP-WIN`
5. Lõpeta paigaldus
6. Käivita `services.msc` → **Wazuh** → **Start**

### Windows konfiguratsioon

Fail: `C:\Program Files (x86)\ossec-agent\ossec.conf`

```xml
<client>
  <server>
    <address>10.0.80.10</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

Taaskäivita teenus:
```powershell
Restart-Service WazuhSvc
```

---

# 5. Probleemide lahendamine

## 5.1 Paigaldus ebaõnnestub

### "Cannot install dependency"

```bash
# 1. Puhasta apt
sudo apt clean
sudo rm -rf /var/lib/apt/lists/*
sudo apt update

# 2. Paranda katkised
sudo apt --fix-broken install -y
sudo dpkg --configure -a

# 3. Paigalda puuduvad
sudo apt install -y software-properties-common gnupg2 curl

# 4. Proovi uuesti
sudo bash wazuh-install.sh -a -i
```

### "UFW enabled" hoiatus

```bash
# Ava pordid
sudo ufw allow 443/tcp
sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 9200/tcp
sudo ufw allow 55000/tcp

# VÕI keela ajutiselt
sudo ufw disable
# Paigalda Wazuh
sudo bash wazuh-install.sh -a -i
# Luba tagasi
sudo ufw enable
```

## 5.2 Paigalduse puhastamine ja uuesti alustamine

```bash
# 1. Peata teenused
sudo systemctl stop wazuh-manager wazuh-indexer wazuh-dashboard 2>/dev/null

# 2. Eemalda paketid
sudo apt remove --purge wazuh-manager wazuh-indexer wazuh-dashboard filebeat -y 2>/dev/null
sudo apt autoremove -y

# 3. Kustuta kaustad
sudo rm -rf /var/ossec
sudo rm -rf /usr/share/wazuh-*
sudo rm -rf /etc/wazuh-*
sudo rm -rf /var/lib/wazuh-*
sudo rm -rf /run/wazuh-*

# 4. Kustuta skriptid
cd /tmp
rm -f wazuh-install.sh config.yml wazuh-install-files.tar

# 5. Alusta uuesti
curl -sO https://packages.wazuh.com/4.10/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.10/config.yml
# Muuda config.yml ja käivita
sudo bash wazuh-install.sh -a -i
```

## 5.3 Indexer ei käivitu

```bash
# Kontrolli logi
sudo journalctl -u wazuh-indexer -n 50

# Tavaline probleem: vähe mälu
free -h

# Lisa swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Taaskäivita
sudo systemctl restart wazuh-indexer
```

## 5.4 Dashboard ei avane

```bash
# Kontrolli teenust
sudo systemctl status wazuh-dashboard

# Kontrolli porti
sudo ss -tlnp | grep 443

# Taaskäivita
sudo systemctl restart wazuh-dashboard

# Vaata logi
sudo journalctl -u wazuh-dashboard -n 50
```

## 5.5 Agent ei ühendu

**Serveris:**
```bash
# Kontrolli agente
sudo /var/ossec/bin/agent_control -l

# Kontrolli porti
sudo ss -tlnp | grep 1514

# Vaata logi
sudo tail -f /var/ossec/logs/ossec.log
```

**Agendis:**
```bash
# Kontrolli ühendust
nc -zv WAZUH_SERVER_IP 1514

# Kontrolli teenust
sudo systemctl status wazuh-agent

# Vaata logi
sudo tail -f /var/ossec/logs/ossec.log
```

---

## Vigade tabel

| Probleem | Põhjus | Lahendus |
|----------|--------|----------|
| Cannot install dependency | apt cache vananenud | `sudo apt update && sudo apt --fix-broken install` |
| UFW enabled warning | Tulemüür blokeerib | Ava pordid 443, 1514, 1515, 9200, 55000 |
| Indexer fails to start | Vähe mälu | Lisa 4GB swap |
| Dashboard 502 | Indexer maas | `sudo systemctl restart wazuh-indexer` |
| Agent disconnected | Vale IP `ossec.conf`-is | Kontrolli `<address>` väärtust |
| Password not shown | Väljund kadus | `cat /tmp/wazuh-install-files/wazuh-passwords.txt` |

---

## Kasulikud käsud

```bash
# Teenuste haldamine
sudo systemctl restart wazuh-manager wazuh-indexer wazuh-dashboard
sudo systemctl status wazuh-manager wazuh-indexer wazuh-dashboard

# Agentide nimekiri
sudo /var/ossec/bin/agent_control -l

# Logid
sudo tail -f /var/ossec/logs/ossec.log
sudo journalctl -u wazuh-indexer -f
sudo journalctl -u wazuh-dashboard -f

# Paigalduse logi
sudo cat /var/log/wazuh-install.log

# Versioon
sudo /var/ossec/bin/wazuh-control info
```

---

## Kasulikud lingid

- [Wazuh ametlik dokumentatsioon](https://documentation.wazuh.com/current/index.html)
- [Wazuh All-in-One paigaldus](https://documentation.wazuh.com/current/installation-guide/wazuh-server/installation-assistant.html)
- [Wazuh Agent paigaldus](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/index.html)
- [Wazuh Windows Agent](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
