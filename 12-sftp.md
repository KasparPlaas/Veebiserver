# SFTP
[Tagasi README](README.md) · [← Eelmine](11-ftp-vsftpd.md) · [Järgmine →](13-samba.md)

## Server (Debian)
```bash
sudo apt install ssh
sudo groupadd sftpgroup
sudo useradd -G sftpgroup -d /srv/sftpuser -s /sbin/nologin sftpuser
sudo passwd sftpuser
sudo mkdir -p /srv/sftpuser/data
sudo chown root /srv/sftpuser
sudo chmod g+rx /srv/sftpuser
sudo chown sftpuser:sftpuser /srv/sftpuser/data

sudo nano /etc/ssh/sshd_config
# Subsystem sftp internal-sftp
# Match Group sftpgroup
#   ChrootDirectory %h
#   X11Forwarding no
#   AllowTCPForwarding no
#   ForceCommand internal-sftp

sudo systemctl restart sshd
```

## Klient
```bash
# AlmaLinux
sudo dnf install ssh
sftp sftpuser@masin
# Debian
sudo apt install ssh
sftp sftpuser@masin
```

## Vigade leidmine ja parandamine
- Chroot õigused: juurkaust peab kuuluma `root`

## Levinud vead
- `Subsystem` vale rida (kommenteerimata vana)
