# FTP Server (vsftpd)

> vsftpd (Very Secure FTP Daemon) on turvaline ja kiire FTP server. See juhend näitab põhiseadistust failide jagamiseks.

[Tagasi README](README.md) · [← Eelmine](10-tftp.md) · [Järgmine →](12-sftp.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Port 21 on tulemüüris lubatud

---

## Paigaldamine

```bash
sudo apt update
sudo apt install vsftpd -y
```

---

## Kausta ja grupi seadistamine

```bash
sudo mkdir -p /srv/ftp
sudo groupadd ftpuser
sudo chown root:ftpuser /srv/ftp
sudo chmod 775 /srv/ftp
sudo usermod -aG ftpuser kasutaja
```

---

## Konfigureerimine

```bash
sudo nano /etc/vsftpd.conf
```

Olulised seaded:
```text
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
local_root=/srv/ftp
chroot_local_user=YES
allow_writeable_chroot=YES
```

Teenuse taaskäivitamine:
```bash
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd
```

---

## DNS kirje (valikuline)

Lisa DNS tsooni:
```text
ftp     IN      A       10.0.80.2
```

---

## Tulemüür

```bash
sudo ufw allow 21/tcp
sudo ufw allow 20/tcp
```

Passiivse režiimi jaoks:
```bash
sudo ufw allow 40000:50000/tcp
```

Lisa `vsftpd.conf` faili:
```text
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000
```

---

## Klient

```bash
ftp ftp.plaas.lan
```

või
```bash
ftp 10.0.80.2
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Login ebaõnnestub | Kontrolli kasutaja õigusi ja gruppi |
| Ei saa kirjutada | `write_enable=YES` ja kausta õigused |
| Passiivne režiim ei tööta | Lisa pasv_* seaded ja ava pordid |

---

## Levinud vead

- **Kausta õigused valed** – kontrolli `/srv/ftp` omanikku ja gruppe
- **PASSIVE pordid puudu** – kliendid NAT taga vajavad passiivset režiimi
- **Chroot viga** – `allow_writeable_chroot=YES` kui kaust on kirjutatav
