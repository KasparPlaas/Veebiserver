# UFW Tulemüür

> UFW (Uncomplicated Firewall) on kasutajasõbralik tulemüür Linuxile. See võimaldab lihtsalt hallata võrgureegleid.

[Tagasi README](README.md) · [← Eelmine](04-dhcp.md) · [Järgmine →](06-apache-https.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- SSH ühendus on loodud (oluline mitte ennast välja lukustada!)

---

## Paigaldamine

```bash
sudo apt update
sudo apt install ufw -y
```

---

## Põhiseadistus

```bash
# SSH (oluline lisada enne enable!)
sudo ufw allow ssh

# Levinud teenused
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 53         # DNS
sudo ufw allow 67/udp     # DHCP

# Tulemüüri lubamine
sudo ufw enable
```

---

## Kasulikud käsud

```bash
sudo ufw status verbose      # näita reegleid ja staatust
sudo ufw status numbered     # näita reegleid numbritega
sudo ufw delete 3            # kustuta reegel nr 3
sudo ufw allow from 10.0.80.0/24    # luba terve alamvõrk
sudo ufw deny 23             # blokeeri port
sudo ufw reset               # lähtesta kõik reeglid
```

---

## Kontrollimine

```bash
sudo ufw status
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Port puudu | `sudo ufw allow <port>` |
| Lukustatud välja | Kasuta konsooli ja `sudo ufw disable` |
| Reeglite vaatamine | `sudo ufw status numbered` |

---

## Levinud vead

- **UFW lubamata** – `enable` käsk unustatud peale reeglite lisamist
- **SSH blokeeritud** – enne `enable` käsku lisa `allow ssh`
- **Vale protokoll** – mõned teenused nõuavad UDP-d (nt DHCP port 67/udp)
