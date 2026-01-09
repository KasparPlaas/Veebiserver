# DHCP (isc-dhcp-server)
[Tagasi README](README.md) · [← Eelmine](03-bind9.md) · [Järgmine →](05-ufw.md)

## Install + seadistus
```bash
sudo apt install isc-dhcp-server
sudo nano /etc/default/isc-dhcp-server
# INTERFACES="ens18"

sudo nano /etc/dhcp/dhcpd.conf
# subnet 10.0.80.0 netmask 255.255.255.0 {
#   range 10.0.80.100 10.0.80.200;
#   option domain-name-servers 8.8.8.8;
#   option domain-name "plaas.lan";
#   option routers 10.0.80.1;
#   option broadcast-address 10.0.80.255;
#   default-lease-time 600;
#   max-lease-time 7200;
# }

sudo systemctl start isc-dhcp-server
```

## Kontroll
```bash
sudo systemctl status isc-dhcp-server
``` 

## Vigade leidmine ja parandamine
- Liides vale: kontrolli `INTERFACES` väärtust

## Levinud vead
- Kommentaarid (#) jäetud valesse kohta
