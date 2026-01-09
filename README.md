# Veebiserver Dokumentatsioon

> Praktiline juhendite kogumik Linux serverite seadistamiseks. Sisaldab samm-sammult juhiseid alates baaskonfiguratsioonist kuni keerukate monitooringusüsteemideni.

---

## Baaskonfiguratsioon

Alusta siit! Need juhendid aitavad seadistada serveri algseaded.

| Juhend | Kirjeldus |
|--------|-----------|
| [AlmaLinux Basic Configuration](00-almalinux-alustamine.md) | Hostname, staatiline IP, SSH (RHEL-põhine) |
| [Debian Basic Configuration](01-debian-alustamine.md) | Hostname, staatiline IP, SSH |
| [Ubuntu Basic Configuration](02-ubuntu-alustamine.md) | Hostname, staatiline IP (Netplan), SSH |

---

## Võrguteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [Bind9 DNS](03-bind9.md) | DNS serveri seadistamine |
| [DHCP Server](04-dhcp.md) | Automaatne IP jagamine |
| [UFW Tulemüür](05-ufw.md) | Tulemüüri haldamine |

---

## Veebiserverid

| Juhend | Kirjeldus |
|--------|-----------|
| [Apache HTTPS + Virtual Hosts](06-apache-https.md) | Apache2, SSL sertifikaadid, vhostid |
| [AlmaLinux + NGINX](07-almalinux-staatiline-ip.md) | AlmaLinux IP, firewalld, NGINX |
| [NGINX Virtual Hosts](08-nginx.md) | NGINX konfiguratsioon |

---

## Meiliteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [Postfix SMTP](09-smtp-postfix.md) | Meiliserveri seadistamine |

---

## Failijagamine

| Juhend | Kirjeldus |
|--------|-----------|
| [TFTP](10-tftp.md) | PXE boot ja konfiguratsioonide jagamine |
| [FTP (vsftpd)](11-ftp-vsftpd.md) | FTP server |
| [SFTP](12-sftp.md) | Turvaline failiedastus SSH kaudu |
| [SAMBA](13-samba.md) | Windows/Linux failijagamine |
| [NFS](14-nfs.md) | Linux-Linux failijagamine |

---

## Võrgu- ja salvestusteenused

| Juhend | Kirjeldus |
|--------|-----------|
| [SQUID Proxy](15-squid.md) | Proxy server ja veebifiltreerimine |
| [iSCSI](16-iscsi.md) | Blokkseadmete jagamine üle võrgu |
| [NTP](17-ntp.md) | Ajasünkroniseerimine |

---

## Monitooring ja turvalisus

| Juhend | Kirjeldus |
|--------|-----------|
| [Zabbix](18-zabbix.md) | Ettevõtte tasemel monitooring |
| [Cacti](19-cacti.md) | SNMP-põhine graafikute monitooring |
| [Nagios](20-nagios.md) | Teenuste ja hostide monitooring |
| [Wazuh SIEM](21-wazuh.md) | Turvamonitooring ja logide analüüs |

---

## Kiirlingid

- [DNS (Bind9)](03-bind9.md)
- [Veebiserver (Apache)](06-apache-https.md)
- [Proxy (Squid)](15-squid.md)
- [Monitooring (Zabbix)](18-zabbix.md)
- [Turvalisus (Wazuh)](21-wazuh.md)

---

Iga juhend sisaldab:
- Eeldusi ja nõudeid
- Samm-sammult paigaldusjuhiseid
- Konfiguratsiooninäiteid
- Vigade leidmise ja parandamise tabeleid
- Levinud vigade loetelu
