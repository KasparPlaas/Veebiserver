# Debian alustamine
[Tagasi README](README.md) · [Järgmine →](00-almalinux-alustamine.md)

## Hostname
```bash
sudo hostnamectl set-hostname srv
sudo nano /etc/hosts
# muuda 127.0.1.1 srv
```

## Staatiline IP
```bash
sudo nano /etc/network/interfaces
```
Näidis:
```text
auto ens18
iface ens18 inet static
    address 10.0.80.2
    netmask 255.255.255.0
    gateway 10.0.80.1
    dns-nameservers 8.8.8.8
```
```bash
sudo systemctl restart networking
# või
sudo ifdown ens18 && sudo ifup ens18
```

## OpenSSH
```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

## Kontroll
```bash
hostnamectl
ip a
systemctl status ssh
```

## Vigade leidmine ja parandamine
- Võrk ei tule üles: kontrolli liidese nime `ip link`
- SSH ei tööta: `journalctl -u ssh`

## Levinud vead
- Vale liidese nimi `interfaces` failis
- Firewall blokeerib port 22
