# Zabbix Monitooring

> Zabbix on ettevõtte tasemel monitooringulahendus, mis võimaldab jälgida servereid, võrguseadmeid ja rakendusi reaalajas.

<p align="center">
  <a href="18-ntp.md"><img src="https://img.shields.io/badge/Eelmine-NTP-00BCD4?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="20-cacti.md"><img src="https://img.shields.io/badge/Järgmine-Cacti-689F38?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian 12/13 server on seadistatud
- Vähemalt 2GB RAM ja 10GB kettaruumi
- Kasutajal on sudo õigused

---

## Zabbix Serveri paigaldamine

### Repository lisamine

```bash
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian12_all.deb
sudo dpkg -i zabbix-release_latest_7.0+debian12_all.deb
sudo apt update
```

### Pakettide paigaldamine

```bash
sudo apt install -y zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server
```

### MariaDB seadistamine

```bash
sudo mysql -uroot
```

SQL käsud:
```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'tugev_parool';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

### Andmebaasi skeemi import

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

Keela funktsioonide loomise õigus:
```bash
sudo mysql -uroot -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

### Zabbix serveri konfigureerimine

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Seadista:
```text
DBPassword=tugev_parool
```

### Teenuste käivitamine

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

### Locale (vajadusel)

```bash
sudo apt install locales
sudo dpkg-reconfigure locales
```

---

## Veebiinterfeis

Ava brauseris: `http://server-ip/zabbix`

Esmasel sisselogimisel:
- **Username:** Admin
- **Password:** zabbix

---

## Zabbix Agent (AlmaLinux)

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/7.0/rhel/9/x86_64/zabbix-release-latest-7.0.el9.noarch.rpm
sudo dnf clean all
sudo dnf install zabbix-agent -y
```

```bash
sudo nano /etc/zabbix/zabbix_agentd.conf
```

Seadista:
```text
Server=zabbix-server-ip
ServerActive=zabbix-server-ip
Hostname=alma-klient
```

```bash
sudo systemctl enable --now zabbix-agent
sudo firewall-cmd --permanent --add-port=10050/tcp
sudo firewall-cmd --reload
```

---

## Zabbix Agent (Windows)

1. Laadi alla: https://www.zabbix.com/download_agents
2. Paigalda MSI pakett
3. Seadista Zabbix serveri IP
4. Käivita teenus

---

## Hostide lisamine

1. **Configuration** → **Hosts** → **Create host**
2. Sisesta hostname ja IP
3. Vali sobiv template (nt "Linux by Zabbix agent")
4. Lisa gruppi
5. Salvesta

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| DB ühenduse viga | Kontrolli parooli `zabbix_server.conf` failis |
| Agent ei ühendu | Kontrolli tulemüüri (port 10050) |
| Frontend ei tööta | `systemctl status apache2` |

---

## Levinud vead

- **log_bin_trust_function_creators** – peab olema 1 andmebaasi importimisel
- **Tulemüür** – port 10050 peab olema avatud agentidele
- **Vale hostname** – agendi hostname peab vastama Zabbixis määratule
