# NFS
[Tagasi README](README.md) · [← Eelmine](13-samba.md) · [Järgmine →](15-squid.md)

## Server
```bash
sudo apt install nfs-kernel-server
sudo mkdir /srv/nfs && sudo chmod 777 /srv/nfs
sudo nano /etc/exports
# /srv/nfs 10.0.80.2/24(rw,nohide,insecure,no_subtree_check,async)
sudo systemctl restart nfs-kernel-server
sudo ufw allow 111,2049/tcp
sudo ufw allow 111,2049/udp
```

## Klient
```bash
# AlmaLinux
sudo dnf install nfs-utils -y
# Debian
sudo apt install nfs-common -y
# Mount
sudo mount -t nfs -o proto=tcp,port=2049 10.0.80.2:/ /mnt
```

## Vigade leidmine ja parandamine
- Export puudu: `exportfs -v`

## Levinud vead
- Firewall blokeerib 2049
