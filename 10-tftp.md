# TFTP Server (tftpd-hpa)

> TFTP on lihtne failiedastusprotokoll, mida kasutatakse peamiselt võrguseadmete alglaadimiseks (PXE boot) ja konfiguratsioonifailide jagamiseks.

<p align="center">
  <a href="09-smtp-postfix.md"><img src="https://img.shields.io/badge/Eelmine-Postfix_SMTP-005FF9?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="11-ftp-vsftpd.md"><img src="https://img.shields.io/badge/Järgmine-FTP-FF6F00?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Port 69/UDP on tulemüüris lubatud

---

## Paigaldamine

```bash
sudo apt update
sudo apt install tftpd-hpa -y
```

---

## Konfigureerimine

```bash
sudo nano /etc/default/tftpd-hpa
```

Seadistus:
```text
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure --create"
```

Teenuse taaskäivitamine:
```bash
sudo systemctl restart tftpd-hpa
sudo systemctl enable tftpd-hpa
```

---

## Kaustade struktuur

```bash
sudo mkdir -p /srv/tftp/openbsd
sudo mkdir -p /srv/tftp/freebsd
sudo mkdir -p /srv/tftp/netbsd
sudo mkdir -p /srv/tftp/linux/{debian,ubuntu,rhel,centos,fedora,suse}
```

### Näide: OpenBSD PXE failid

```bash
cd /srv/tftp/openbsd/
sudo wget http://ftp.openbsd.org/pub/OpenBSD/7.7/i386/pxeboot
sudo wget http://ftp.openbsd.org/pub/OpenBSD/7.7/i386/bsd.rd
```

---

## TFTP klient

### Debian
```bash
sudo apt install tftp-hpa
```

### AlmaLinux
```bash
sudo dnf install tftp
```

### Kasutamine
```bash
tftp <SERVER_IP>
tftp> get filename
tftp> put filename
tftp> quit
```

---

## Tulemüür

### UFW
```bash
sudo ufw allow 69/udp
```

### Firewalld (AlmaLinux)
```bash
sudo firewall-cmd --permanent --add-service=tftp
sudo firewall-cmd --reload
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Faili ei leia | Kontrolli õigusi kaustal `/srv/tftp` |
| Ühendus ebaõnnestub | Kontrolli port 69/UDP tulemüüris |
| Permission denied | `sudo chown -R tftp:tftp /srv/tftp` |

---

## Levinud vead

- **Vale port** – TFTP kasutab porti 69/UDP
- **Kausta õigused** – `/srv/tftp` peab olema loetav (ja kirjutatav, kui `--create`)
- **SELinux** – AlmaLinuxil võib blokeerida ligipääsu
