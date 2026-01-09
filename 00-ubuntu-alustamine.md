# Ubuntu alustamine
[Tagasi README](README.md) · [← Eelmine](00-almalinux-alustamine.md) · [Järgmine →](01-debian-staatiline-ip.md)

## Hostname
```bash
sudo hostnamectl set-hostname srv
sudo nano /etc/hosts
# muuda 127.0.1.1 srv
```

## Staatiline IP (Netplan)
```bash
sudo nano /etc/netplan/01-netcfg.yaml
```
Näidis:
```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.0.80.4/24
      gateway4: 10.0.80.1
      nameservers:
        addresses:
          - 8.8.8.8
```
```bash
sudo netplan apply
```

## OpenSSH
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo ufw allow ssh
sudo ufw enable
```

## Kontroll
```bash
hostnamectl
ip a
systemctl status ssh
```

## Vigade leidmine ja parandamine
- Netplan viga: `sudo netplan try` (testimiseks)
- SSH ei tööta: `sudo ufw status`

## Levinud vead
- YAML taande viga (kasuta tühikuid, mitte tabulaatoreid)
- UFW pole lubatud
