# Bind9 (DNS)
[Tagasi README](README.md) · [← Eelmine](02-debian-uuendus.md) · [Järgmine →](04-dhcp.md)

## Install + forwarder
```bash
sudo apt install bind9
sudo nano /etc/bind/named.conf.options
# lisa:
# options {
#   directory "/var/cache/bind";
#   forwarders { 193.40.56.245; };
#   auth-nxdomain no;
#   listen-on-v6 { any; };
# };
```

## Tsoonifail
```bash
sudo cp /etc/bind/db.local /etc/bind/db.plaas.lan
# sisu (näidis):
# @ IN SOA ns.plaas.lan. root.plaas.lan. (
#   3 604800 86400 2419200 604800 )
# @ IN NS ns.plaas.lan.
# @ IN A 10.0.80.2
# ns.plaas.lan. IN A 10.0.80.2
# plaas.lan. IN A 10.0.80.2
# www.plaas.lan. IN A 10.0.80.3

sudo nano /etc/bind/named.conf.local
# zone "plaas.lan" IN {
#   type master;
#   file "/etc/bind/db.plaas.lan";
# };
```

## Kontroll
```bash
named-checkzone plaas.lan /etc/bind/db.plaas.lan
sudo systemctl restart bind9
dig @localhost www.plaas.lan
```

## Vigade leidmine ja parandamine
- `Serial` muutmata: tõsta numbrit + restart
- Syntaks: `named-checkconf` ja `journalctl -u bind9`

## Levinud vead
- Punkti unustamine FQDN lõpus (ns.plaas.lan.)
