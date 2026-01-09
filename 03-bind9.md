# Bind9 DNS Server

> BIND9 on kõige laialdasemalt kasutatav DNS server tarkvara. See juhend näitab, kuidas seadistada oma DNS serverit.

[Tagasi README](README.md) · [← Eelmine](02-ubuntu-alustamine.md) · [Järgmine →](04-dhcp.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Staatiline IP-aadress on konfigureeritud
- Kasutajal on sudo õigused

---

## Paigaldamine

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

---

## Forwarder seadistamine

Forwarder suunab päringud edasi teisele DNS serverile, kui kohalik server vastust ei tea.

```bash
sudo nano /etc/bind/named.conf.options
```

Sisu:
```text
options {
    directory "/var/cache/bind";
    forwarders {
        193.40.56.245;
        8.8.8.8;
    };
    dnssec-validation auto;
    auth-nxdomain no;
    listen-on-v6 { any; };
};
```

---

## Tsoonifaili loomine

### Tsoonifaili kopeerimine ja muutmine

```bash
sudo cp /etc/bind/db.local /etc/bind/db.plaas.lan
sudo nano /etc/bind/db.plaas.lan
```

Näidissisu:
```text
$TTL    604800
@       IN      SOA     ns.plaas.lan. root.plaas.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.plaas.lan.
@       IN      A       10.0.80.2
ns      IN      A       10.0.80.2
www     IN      A       10.0.80.3
```

> **Oluline:** Serial numbrit tuleb iga muudatuse järel suurendada!

### Tsooni registreerimine

```bash
sudo nano /etc/bind/named.conf.local
```

Lisa:
```text
zone "plaas.lan" IN {
    type master;
    file "/etc/bind/db.plaas.lan";
};
```

---

## Kontrollimine

```bash
named-checkconf                                    # konfiguratsioon
named-checkzone plaas.lan /etc/bind/db.plaas.lan   # tsoonifail
sudo systemctl restart bind9
dig @localhost www.plaas.lan                       # testimine
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Muudatused ei rakendu | Tõsta Serial numbrit ja tee restart |
| Süntaksi vead | `named-checkconf` ja `named-checkzone` |
| Logide vaatamine | `journalctl -u bind9` |

---

## Levinud vead

- **Punkti unustamine FQDN lõpus** – `ns.plaas.lan` asemel peab olema `ns.plaas.lan.`
- **Serial numbri muutmata jätmine** – muudatused ei rakendu
- **Vale failitee** – kontrolli `named.conf.local` failis olevat teed
