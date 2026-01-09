# SFTP Server

> SFTP (SSH File Transfer Protocol) on turvaline failiedastusprotokoll, mis kasutab SSH krüpteeringut. See on turvalisem alternatiiv tavalisele FTP-le.

[Tagasi README](README.md) · [← Eelmine](11-ftp-vsftpd.md) · [Järgmine →](13-samba.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- OpenSSH server on paigaldatud
- Kasutajal on sudo õigused

---

## Serveri seadistamine (Debian)

### SFTP grupi ja kasutaja loomine

```bash
sudo groupadd sftpgroup
sudo useradd -G sftpgroup -d /srv/sftpuser -s /sbin/nologin sftpuser
sudo passwd sftpuser
```

### Kaustade seadistamine

```bash
sudo mkdir -p /srv/sftpuser/data
sudo chown root:root /srv/sftpuser
sudo chmod 755 /srv/sftpuser
sudo chown sftpuser:sftpuser /srv/sftpuser/data
```

> **Oluline:** Chroot kaust peab kuuluma `root` kasutajale ja ei tohi olla kirjutatav teiste poolt.

---

## SSH konfigureerimine

```bash
sudo nano /etc/ssh/sshd_config
```

Lisa faili lõppu:
```text
# SFTP seadistus
Subsystem sftp internal-sftp

Match Group sftpgroup
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
```

> **NB!** Kui `Subsystem sftp` rida on juba olemas, kommenteeri vana välja.

Teenuse taaskäivitamine:
```bash
sudo systemctl restart sshd
```

---

## SFTP klient

### Debian
```bash
sudo apt install openssh-client
sftp sftpuser@server-ip
```

### AlmaLinux
```bash
sudo dnf install openssh-clients
sftp sftpuser@server-ip
```

### Kasulikud käsud SFTP sessioonis
```text
ls              - kausta sisu
cd              - muuda kausta
get fail        - laadi alla
put fail        - laadi üles
mkdir kaust     - loo kaust
exit            - välju
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Login ebaõnnestub | Kontrolli kasutaja gruppi |
| Chroot viga | Juurkaust peab kuuluma `root` kasutajale |
| "Write failed" | Kontrolli `/data` kausta õigusi |

---

## Levinud vead

- **Chroot kausta õigused** – `/srv/sftpuser` peab kuuluma `root:root` ja olema `755`
- **Vale Subsystem rida** – vana rida peab olema kommenteeritud
- **Grupp vale** – kasutaja peab olema `sftpgroup` grupis
