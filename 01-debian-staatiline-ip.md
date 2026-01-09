# Debian staatiline IP + hostnimi
[Tagasi README](README.md) · [Järgmine →](02-debian-uuendus.md)

## Sammud
```bash
sudo nano /etc/network/interfaces
# salvesta
sudo systemctl restart networking
# või
sudo ifdown <liides> && sudo ifup <liides>

# Hostname
sudo hostnamectl set-hostname srv
sudo newgrp
sudo nano /etc/hosts
# muuda 127.0.1.1 srv

# Peegli muutmine
sudo sed -i -e 's/deb.debian.org/mirrors.xtom.ee/g' /etc/apt/sources.list

# Uuendus
sudo apt update && sudo apt upgrade
```

## Kontroll
- `hostnamectl` näitab `srv`
- `ip a` näitab õiget IP

## Vigade leidmine ja parandamine
- Võrgu restart ei toimi: kontrolli liidese nime (`ip link`)
- `hosts` vale: ping `srv` peaks lahenduma localhostile

## Levinud vead
- Tippvead `interfaces` failis
- Unustatakse `systemctl restart networking`
