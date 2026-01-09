# Nagios Monitooring

> Nagios on võimas avatud lähtekoodiga monitooringusüsteem, mis jälgib servereid, teenuseid ja võrguseadmeid ning teavitab probleemidest.

<p align="center">
  <a href="20-cacti.md"><img src="https://img.shields.io/badge/Eelmine-Cacti-689F38?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="22-wazuh.md"><img src="https://img.shields.io/badge/Järgmine-Wazuh_SIEM-3CBCE8?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Apache2 on paigaldatud
- Kasutajal on sudo õigused

---

## Paigaldamine

```bash
sudo apt update
sudo apt install -y nagios4 nagios-plugins nagios-nrpe-plugin
```

---

## CGI seadistamine

### Autentimise lubamine

```bash
sudo nano /etc/nagios4/cgi.cfg
```

Seadista:
```text
use_authentication=1
```

### Apache konfigureerimine

```bash
sudo nano /etc/apache2/conf-enabled/nagios4-cgi.conf
```

Lisa/muuda:
```apache
<Directory "/usr/lib/cgi-bin/nagios4">
    Options ExecCGI
    AllowOverride None
    
    AuthType Digest
    AuthName "Nagios4"
    AuthDigestDomain "/nagios4"
    AuthUserFile "/etc/nagios4/htdigest.users"
    Require valid-user
</Directory>
```

### Kasutaja loomine

```bash
sudo htdigest -c /etc/nagios4/htdigest.users "Nagios4" nagiosadmin
```

### Mooduli lubamine

```bash
sudo a2enmod cgid auth_digest
sudo systemctl restart nagios4 apache2
```

---

## Veebiinterfeis

Ava brauseris: `http://server-ip/nagios4`

---

## Hostide lisamine

### Kausta loomine

```bash
sudo mkdir -p /etc/nagios4/servers
```

Lisa `nagios.cfg` faili:
```bash
sudo nano /etc/nagios4/nagios.cfg
```

```text
cfg_dir=/etc/nagios4/servers
```

### Linux hosti näide (AlmaLinux)

```bash
sudo nano /etc/nagios4/servers/alma.cfg
```

```text
define host {
    use                     linux-server
    host_name               alma01
    alias                   AlmaLinux Server
    address                 10.0.80.3
    max_check_attempts      5
    check_period            24x7
    notification_interval   30
    notification_period     24x7
}

define service {
    use                     generic-service
    host_name               alma01
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}

define service {
    use                     generic-service
    host_name               alma01
    service_description     SSH
    check_command           check_ssh
}
```

### Windows hosti näide

```bash
sudo nano /etc/nagios4/servers/windows.cfg
```

```text
define host {
    use                     windows-server
    host_name               win01
    alias                   Windows Client
    address                 10.0.80.100
}

define service {
    use                     generic-service
    host_name               win01
    service_description     PING
    check_command           check_ping!100.0,20%!500.0,60%
}
```

---

## Konfiguratsiooni kontrollimine

```bash
sudo nagios4 -v /etc/nagios4/nagios.cfg
```

Kui vigu pole:
```bash
sudo systemctl restart nagios4
```

---

## NRPE Agent (kaugmonitoring)

### Agendi paigaldamine (Debian kliendil)

```bash
sudo apt install -y nagios-nrpe-server nagios-plugins
```

```bash
sudo nano /etc/nagios/nrpe.cfg
```

Seadista:
```text
allowed_hosts=127.0.0.1,nagios-server-ip
```

```bash
sudo systemctl restart nagios-nrpe-server
```

### Tulemüür

```bash
sudo ufw allow 5666/tcp
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Konfiviga | `nagios4 -v /etc/nagios4/nagios.cfg` |
| Auth ei tööta | Kontrolli htdigest faili ja Apache moodulit |
| Host punane | Kontrolli võrguühendust ja tulemüüri |

---

## Levinud vead

- **Vale check_command** – parameetrid peavad vastama plugin nõuetele
- **cfg_dir puudu** – `/etc/nagios4/servers` peab olema lisatud
- **Õigused valed** – konfifailid peavad olema loetavad nagios kasutajale
