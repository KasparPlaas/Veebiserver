# FTP (vsftpd)
[Tagasi README](README.md) · [← Eelmine](10-tftp.md) · [Järgmine →](12-sftp.md)

## Install + kaust
```bash
sudo apt install vsftpd
sudo mkdir /srv/ftp
sudo groupadd ftpuser
sudo chown :ftpuser /srv/ftp
sudo chmod g+w /srv/ftp
sudo usermod -aG ftpuser kasutaja
```

## Konf
```bash
sudo nano /etc/vsftpd.conf
# listen=YES
# local_enable=YES
# write_enable=YES
# local_root=/srv/ftp
sudo systemctl restart vsftpd
```

## DNS + firewall
```text
ftp.minu.lan. IN A x.x.x.x
```
```bash
sudo ufw allow 21
```

## Klient
```bash
ftp ftp.minu.lan
```

## Vigade leidmine ja parandamine
- Õigused kaustal `/srv/ftp`

## Levinud vead
- PASSIVE portide puudumine suuremates seadistustes
