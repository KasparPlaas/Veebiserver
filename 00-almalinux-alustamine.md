# AlmaLinux alustamine
[Tagasi README](README.md) · [← Eelmine](00-debian-alustamine.md) · [Järgmine →](00-ubuntu-alustamine.md)

## Hostname
```bash
sudo hostnamectl set-hostname srv
sudo nano /etc/hosts
# lisa 127.0.1.1 srv
```

## Staatiline IP
```bash
sudo nmtui
```
- Edit a connection → vali liides
- IPv4: Manual → lisa IP, gateway, DNS
- OK → Back → Activate a connection → deactivate/activate

Või käsurealt:
```bash
sudo nmcli con mod ens18 ipv4.addresses 10.0.80.3/24
sudo nmcli con mod ens18 ipv4.gateway 10.0.80.1
sudo nmcli con mod ens18 ipv4.dns "8.8.8.8"
sudo nmcli con mod ens18 ipv4.method manual
sudo nmcli con up ens18
```

## OpenSSH
```bash
sudo dnf install openssh-server -y
sudo systemctl enable --now sshd
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

## Kontroll
```bash
hostnamectl
ip a
systemctl status sshd
```

## Vigade leidmine ja parandamine
- Võrk ei tule: `nmcli con show` ja kontrolli seadeid
- SSH ei tööta: `firewall-cmd --list-services`

## Levinud vead
- Unustatud firewalld reeglid
- `nmtui` muudatusi ei aktiveerita
