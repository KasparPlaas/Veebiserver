# Debian 12 → 13 uuendus
[Tagasi README](README.md) · [← Eelmine](01-debian-staatiline-ip.md) · [Järgmine →](03-bind9.md)

## Sammud
```bash
sudo apt update && sudo apt upgrade
sudo apt dist-upgrade
```
Viide: https://linuxconfig.org/how-to-upgrade-debian-to-latest-version

## Kontroll
- `lsb_release -a` või `/etc/os-release` näitab uut versiooni

## Vigade leidmine ja parandamine
- Lukud APT-s: `sudo rm /var/lib/apt/lists/lock; sudo dpkg --configure -a`

## Levinud vead
- Ebastabiilsed repo kirjed `sources.list`
