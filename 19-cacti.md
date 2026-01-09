# Cacti
[Tagasi README](README.md) · [← Eelmine](18-zabbix.md) · [Järgmine →](20-nagios.md)

## Repo + install
```bash
sudo nano /etc/apt/sources.list
# deb http://deb.debian.org/debian/ trixie main non-free-firmware non-free
# deb http://security.debian.org/debian-security trixie-security main non-free-firmware non-free
# deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware non-free
# deb http://deb.debian.org/debian/ trixie-backports main non-free-firmware non-free

sudo apt update
sudo apt -y install cacti snmp snmpd snmp-mibs-downloader php-mysql php-snmp php-intl rrdtool
```

## SNMPD
```bash
sudo nano /etc/snmp/snmpd.conf
# agentAddress udp:161
# rocommunity public 10.0.80.0/24
# sysLocation Serveriruum
# sysContact admin@example.com
sudo ufw allow 161 && sudo systemctl restart snmpd && sudo systemctl enable snmpd
```
Test:
```bash
snmpwalk -v2c -c public 10.0.80.2
```

## Windows SNMP
```powershell
Add-WindowsCapability -Online -Name SNMP.Client~~~~0.0.1.0
```
Seaded: Security → community `public` (RO), lubatud hostid → 10.0.80.2.

## Cacti UI
`https://<ip>/cacti` → `admin/Passw0rd`. Lisa seadmed: Debian (10.0.80.2), Windows (10.0.80.100).

## Vigade leidmine ja parandamine
- SNMP community/ACL vale

## Levinud vead
- 161/tcp vs 161/udp segamini
