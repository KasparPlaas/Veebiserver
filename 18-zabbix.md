# Zabbix
[Tagasi README](README.md) · [← Eelmine](17-ntp.md) · [Järgmine →](19-cacti.md)

## Server
```bash
wget https://repo.zabbix.com/zabbix/7.4/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.4+debian13_all.deb
sudo dpkg -i zabbix-release_latest_7.4+debian13_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent mariadb-server

sudo mysql -uroot <<'SQL'
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
SQL

zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
sudo mysql -uroot -e 'set global log_bin_trust_function_creators = 0;'

sudo nano /etc/zabbix/zabbix_server.conf
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

Locale:
```bash
sudo apt install locales
sudo dpkg-reconfigure locales
```

UI: mine `http://<ip>/zabbix` ja lõpeta seadistus.

## Agendid
AlmaLinux:
```bash
rpm -Uvh https://repo.zabbix.com/zabbix/6.0/rhel/9/x86_64/zabbix-release-latest-6.0.el9.noarch.rpm
dnf clean all
dnf install zabbix-agent
sudo nano /etc/zabbix/zabbix_agentd.conf
# Server=<zabbix_server_ip>
sudo systemctl restart zabbix-agent && sudo systemctl enable zabbix-agent
```
Windows: lae alla installer → pane serveri IP.

## Hostide lisamine
Monitoring → Hosts → Create host → vali template, grupp, lisa agenti IP.

## Vigade leidmine ja parandamine
- MariaDB õigused: kontrolli kasutajat/parooli
- Agent ei nähtu: firewall

## Levinud vead
- Unustatud `log_bin_trust_function_creators`
