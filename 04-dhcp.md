# DHCP Server (isc-dhcp-server)

> DHCP server jagab automaatselt IP-aadresse võrgus olevatele seadmetele. See lihtsustab võrgu haldamist oluliselt.

<p align="center">
  <a href="03-bind9.md"><img src="https://img.shields.io/badge/Eelmine-Bind9_DNS-4285F4?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="05-ufw.md"><img src="https://img.shields.io/badge/Järgmine-UFW-DD4814?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Serveril on staatiline IP-aadress
- Kasutajal on sudo õigused

---

## Paigaldamine

```bash
sudo apt update
sudo apt install isc-dhcp-server -y
```

---

## Võrguliidese määramine

```bash
sudo nano /etc/default/isc-dhcp-server
```

Määra liides, millel DHCP kuulab:
```text
INTERFACESv4="ens18"
```

---

## Põhikonfiguratsioon

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

Näidiskonfiguratsioon:
```text
# Globaalsed seaded
default-lease-time 600;
max-lease-time 7200;
authoritative;

# Alamvõrgu definitsioon
subnet 10.0.80.0 netmask 255.255.255.0 {
    range 10.0.80.100 10.0.80.200;
    option domain-name-servers 8.8.8.8;
    option domain-name "plaas.lan";
    option routers 10.0.80.1;
    option broadcast-address 10.0.80.255;
}
```

---

## Teenuse käivitamine

```bash
sudo systemctl start isc-dhcp-server
sudo systemctl enable isc-dhcp-server
```

---

## Kontrollimine

```bash
sudo systemctl status isc-dhcp-server
cat /var/lib/dhcp/dhcpd.leases    # vaata väljastatud IP-sid
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Teenus ei käivitu | Kontrolli `INTERFACES` väärtust |
| Klient ei saa IP-d | Kontrolli alamvõrgu seadeid |
| Logide vaatamine | `journalctl -u isc-dhcp-server` |

---

## Levinud vead

- **Vale liides** – `INTERFACES` peab vastama tegelikule võrguliidesele
- **Kommentaarid vales kohas** – konfiguratsioonifaili süntaks peab olema õige
- **Alamvõrk ei vasta serverile** – serveri IP peab olema samas alamvõrgus
