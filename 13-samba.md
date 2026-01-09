# SAMBA jagatud kaust
[Tagasi README](README.md) · [← Eelmine](12-sftp.md) · [Järgmine →](14-nfs.md)

## Server
```bash
sudo apt install samba
sudo mkdir /srv/jagatud
sudo chmod 777 /srv/jagatud
sudo nano /etc/samba/smb.conf
# [jagatud]
# path = /srv/jagatud
# guest ok = yes
# read only = no
# public = yes
# writable = yes
sudo systemctl restart smbd
```

## Klient
Linux:
```bash
sudo apt install smbclient cifs-utils
sudo mkdir /mnt/vorgukaust && sudo chmod 777 /mnt/vorgukaust
sudo mount -t cifs //server-ip/jagatud /mnt/vorgukaust
```
Windows:
- Map network drive → \\server-ip\jagatud

## Vigade leidmine ja parandamine
- Õigused ja \-teed Windowsis

## Levinud vead
- Firewalld/UFW reeglid puudu
