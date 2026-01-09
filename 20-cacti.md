# Cacti Võrgumonitoring

> Cacti on SNMP-põhine võrgumonitooringu tööriist, mis loob ilusaid graafikuid võrguseadmete ja serverite jõudluse kohta.

<p align="center">
  <a href="19-zabbix.md"><img src="https://img.shields.io/badge/Eelmine-Zabbix-D50000?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="21-nagios.md"><img src="https://img.shields.io/badge/Järgmine-Nagios-000000?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian 12/13 server on seadistatud
- Kasutajal on sudo õigused
- LAMP stack (Apache, MySQL, PHP) on paigaldatud

---

## Paigaldamine

### Repository seadistamine

```bash
sudo nano /etc/apt/sources.list
```

Veendu, et on olemas:
```text
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
```

### Pakettide paigaldamine

```bash
sudo apt update
sudo apt install -y cacti snmp snmpd snmp-mibs-downloader php-mysql php-snmp php-intl rrdtool
```

Paigaldamise ajal:
- Vali andmebaasi server: **apache2**
- Seadista Cacti andmebaas: **Yes**
- Sisesta parool

---

## SNMP seadistamine (server)

```bash
sudo nano /etc/snmp/snmpd.conf
```

Seaded:
```text
agentAddress udp:161
rocommunity public 10.0.80.0/24
sysLocation Serveriruum
sysContact admin@plaas.lan
```

```bash
sudo systemctl restart snmpd
sudo systemctl enable snmpd
```

### Tulemüür

```bash
sudo ufw allow 161/udp
```

### Testimine

```bash
snmpwalk -v2c -c public localhost
snmpwalk -v2c -c public 10.0.80.2
```

---

## SNMP (Windows klient)

### PowerShell

```powershell
Add-WindowsCapability -Online -Name SNMP.Client~~~~0.0.1.0
```

### Seadistamine

1. **Services** → **SNMP Service** → **Properties**
2. **Security** tab:
   - Community: `public` (Read Only)
   - Accept SNMP packets: lisa serveri IP
3. Restart teenust

---

## Cacti veebiinterfeis

Ava brauseris: `http://server-ip/cacti`

Esmasel sisselogimisel:
- **Username:** admin
- **Password:** admin (vaheta kohe ära!)

---

## Seadmete lisamine

1. **Console** → **Management** → **Devices** → **Add**
2. Täida:
   - **Description:** Serveri nimi
   - **Hostname:** IP-aadress
   - **SNMP Version:** Version 2
   - **SNMP Community:** public
3. **Create**
4. **Create Graphs for this Host**

---

## Graafikute loomine

1. **Console** → **Management** → **Devices**
2. Vali seade → **Create Graphs**
3. Märgi soovitud graafikud (CPU, Memory, Network)
4. **Create**

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| SNMP ei vasta | Kontrolli community stringi ja ACL-i |
| Graafikud tühjad | Oota 5-10 minutit andmete kogumiseks |
| Permission denied | Kontrolli kausta õigusi |

---

## Levinud vead

- **Community string vale** – peab kattuma serveril ja kliendil
- **Port 161 UDP vs TCP** – SNMP kasutab UDP-d
- **Andmeid pole** – poller peab töötama (kontrolli cron)
