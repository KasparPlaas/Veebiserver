# Veebiserver Dokumentatsioon

> Praktiline juhendite kogumik Linux serverite seadistamiseks. Sisaldab samm-sammult juhiseid alates baaskonfiguratsioonist kuni keerukate monitooringusüsteemideni.

<p align="center">
  <a href="00-almalinux-alustamine.md"><img src="https://img.shields.io/badge/AlmaLinux-EE0000?style=for-the-badge&logo=almalinux&logoColor=white" alt="AlmaLinux"></a>
  <a href="01-debian-alustamine.md"><img src="https://img.shields.io/badge/Debian-A81D33?style=for-the-badge&logo=debian&logoColor=white" alt="Debian"></a>
  <a href="02-ubuntu-alustamine.md"><img src="https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu"></a>
</p>

---

## Baaskonfiguratsioon

Alusta siit! Need juhendid aitavad seadistada serveri algseaded.

| Juhend | Kirjeldus |
|--------|-----------|
| [![AlmaLinux](https://img.shields.io/badge/AlmaLinux_Basic_Config-EE0000?style=flat-square)](00-almalinux-alustamine.md) | Hostname, staatiline IP, SSH (RHEL-põhine) |
| [![Debian](https://img.shields.io/badge/Debian_Basic_Config-A81D33?style=flat-square)](01-debian-alustamine.md) | Hostname, staatiline IP, SSH |
| [![Ubuntu](https://img.shields.io/badge/Ubuntu_Basic_Config-E95420?style=flat-square)](02-ubuntu-alustamine.md) | Hostname, staatiline IP (Netplan), SSH |

---

## Võrguteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [![DNS](https://img.shields.io/badge/Bind9_DNS-4285F4?style=flat-square)](03-bind9.md) | DNS serveri seadistamine |
| [![DHCP](https://img.shields.io/badge/DHCP_Server-00979D?style=flat-square)](04-dhcp.md) | Automaatne IP jagamine |
| [![UFW](https://img.shields.io/badge/UFW_Tulemüür-DD4814?style=flat-square)](05-ufw.md) | Tulemüüri haldamine |

---

## Veebiserverid

| Juhend | Kirjeldus |
|--------|-----------|
| [![Apache](https://img.shields.io/badge/Apache_HTTPS-D22128?style=flat-square&logo=apache&logoColor=white)](06-apache-https.md) | Apache2, SSL sertifikaadid, vhostid |
| [![AlmaLinux](https://img.shields.io/badge/AlmaLinux_+_NGINX-EE0000?style=flat-square)](07-almalinux-staatiline-ip.md) | AlmaLinux IP, firewalld, NGINX |
| [![NGINX](https://img.shields.io/badge/NGINX_Virtual_Hosts-009639?style=flat-square&logo=nginx&logoColor=white)](08-nginx.md) | NGINX konfiguratsioon |

---

## Meiliteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [![Postfix](https://img.shields.io/badge/Postfix_SMTP-005FF9?style=flat-square)](09-smtp-postfix.md) | Meiliserveri seadistamine |

---

## Failijagamine

| Juhend | Kirjeldus |
|--------|-----------|
| [![TFTP](https://img.shields.io/badge/TFTP-607D8B?style=flat-square)](10-tftp.md) | PXE boot ja konfiguratsioonide jagamine |
| [![FTP](https://img.shields.io/badge/FTP_(vsftpd)-FF6F00?style=flat-square)](11-ftp-vsftpd.md) | FTP server |
| [![SFTP](https://img.shields.io/badge/SFTP-231F20?style=flat-square)](12-sftp.md) | Turvaline failiedastus SSH kaudu |
| [![SAMBA](https://img.shields.io/badge/SAMBA-007ACC?style=flat-square)](13-samba.md) | Windows/Linux failijagamine |
| [![Load Balancer](https://img.shields.io/badge/Load_Balancer-FF6B6B?style=flat-square)](14-load-balancer.md) | Koormuse jaotamine (NGINX, HAProxy, Apache) |
| [![NFS](https://img.shields.io/badge/NFS-6DB33F?style=flat-square)](15-nfs.md) | Linux-Linux failijagamine |

---

## Võrgu- ja salvestusteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [![Squid](https://img.shields.io/badge/SQUID_Proxy-EC407A?style=flat-square)](16-squid.md) | Proxy server ja veebifiltreerimine |
| [![iSCSI](https://img.shields.io/badge/iSCSI-795548?style=flat-square)](17-iscsi.md) | Blokkseadmete jagamine üle võrgu |
| [![NTP](https://img.shields.io/badge/NTP-00BCD4?style=flat-square)](18-ntp.md) | Ajasünkroniseerimine |

---

## Monitooring ja turvalisus

| Juhend | Kirjeldus |
|--------|-----------|
| [![Zabbix](https://img.shields.io/badge/Zabbix-D50000?style=flat-square&logo=zabbix&logoColor=white)](19-zabbix.md) | Ettevõtte tasemel monitooring |
| [![Cacti](https://img.shields.io/badge/Cacti-689F38?style=flat-square)](20-cacti.md) | SNMP-põhine graafikute monitooring |
| [![Nagios](https://img.shields.io/badge/Nagios-000000?style=flat-square)](21-nagios.md) | Teenuste ja hostide monitooring |
| [![Wazuh](https://img.shields.io/badge/Wazuh_SIEM-3CBCE8?style=flat-square&logo=wazuh&logoColor=white)](22-wazuh.md) | Turvamonitooring ja logide analüüs |

---

## Kiirlingid

<p align="center">
  <a href="03-bind9.md"><img src="https://img.shields.io/badge/DNS-4285F4?style=for-the-badge" alt="DNS"></a>
  <a href="06-apache-https.md"><img src="https://img.shields.io/badge/Apache-D22128?style=for-the-badge&logo=apache&logoColor=white" alt="Apache"></a>
  <a href="14-load-balancer.md"><img src="https://img.shields.io/badge/Load_Balancer-FF6B6B?style=for-the-badge" alt="Load Balancer"></a>
  <a href="16-squid.md"><img src="https://img.shields.io/badge/Proxy-EC407A?style=for-the-badge" alt="Proxy"></a>
  <a href="19-zabbix.md"><img src="https://img.shields.io/badge/Zabbix-D50000?style=for-the-badge" alt="Zabbix"></a>
  <a href="22-wazuh.md"><img src="https://img.shields.io/badge/Wazuh-3CBCE8?style=for-the-badge" alt="Wazuh"></a>
</p>

---

### Iga juhend sisaldab

<table>
  <tr>
    <td><b>Eeldused</b></td>
    <td>Vajalikud tingimused ja nõuded</td>
  </tr>
  <tr>
    <td><b>Paigaldus</b></td>
    <td>Samm-sammult juhised</td>
  </tr>
  <tr>
    <td><b>Konfiguratsioon</b></td>
    <td>Näidisfailid ja seaded</td>
  </tr>
  <tr>
    <td><b>Veaotsing</b></td>
    <td>Tabelid probleemide lahendamiseks</td>
  </tr>
  <tr>
    <td><b>Levinud vead</b></td>
    <td>Tüüpilised vead ja nende vältimine</td>
  </tr>
</table>
