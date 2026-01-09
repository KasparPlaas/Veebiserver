# TFTP (tftpd-hpa)
[Tagasi README](README.md) · [← Eelmine](09-smtp-postfix.md) · [Järgmine →](11-ftp-vsftpd.md)

## Install + konf
```bash
sudo apt install tftpd-hpa
sudo nano /etc/default/tftpd-hpa
# TFTP_USERNAME="tftp"
# TFTP_DIRECTORY="/srv/tftp"
# TFTP_ADDRESS="0.0.0.0:69"
# TFTP_OPTIONS="--secure"
```
Teenuse restartid:
```bash
sudo systemctl restart tftpd-hpa
```

## Kaustad ja failid
```bash
sudo mkdir -p /srv/tftp/openbsd /srv/tftp/freebsd /srv/tftp/netbsd
sudo mkdir -p /srv/tftp/linux/{debian,ubuntu,rhel,centos,fedora,suse}
cd /srv/tftp/openbsd/
wget http://ftp.openbsd.org/pub/OpenBSD/7.7/i386/pxeboot
wget http://ftp.openbsd.org/pub/OpenBSD/7.7/i386/bsd.rd
```

## Klient
```bash
# Debian
sudo apt install tftp
# AlmaLinux
sudo dnf install tftp
# Ühendus
tftp <SERVER_IP>
```
Firewall (näidis):
```bash
sudo firewall-cmd --add-port=1025-65535/udp --permanent
sudo firewall-cmd --reload
```

## Vigade leidmine ja parandamine
- Õigused: kaust `/srv/tftp` peab olema loetav

## Levinud vead
- Vale port (69/udp)
