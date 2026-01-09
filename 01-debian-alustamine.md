# Debian Basic Configuration

> Debian on stabiilne ja usaldusväärne Linux distributsioon, mida kasutatakse laialdaselt serverites. See juhend aitab seadistada baaskonfiguratsiooni.

<p align="center">
  <a href="00-almalinux-alustamine.md"><img src="https://img.shields.io/badge/Eelmine-AlmaLinux-EE0000?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="02-ubuntu-alustamine.md"><img src="https://img.shields.io/badge/Järgmine-Ubuntu_Basic_Config-E95420?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian 12 (Bookworm) või uuem on paigaldatud
- Kasutajal on sudo õigused
- Võrguühendus on olemas

---

## Hostname seadistamine

Hostname määrab masina identiteedi võrgus.

```bash
sudo hostnamectl set-hostname srv
sudo nano /etc/hosts
```

Muuda või lisa rida:
```text
127.0.1.1   srv
```

---

## Staatiline IP-aadress

Debianis kasutatakse traditsioonilist `/etc/network/interfaces` faili.

```bash
sudo nano /etc/network/interfaces
```

Näidiskonfiguratsioon:
```text
auto ens18
iface ens18 inet static
    address 10.0.80.2
    netmask 255.255.255.0
    gateway 10.0.80.1
    dns-nameservers 8.8.8.8
```

Muudatuste rakendamine:
```bash
sudo systemctl restart networking
```

Alternatiivne meetod:
```bash
sudo ifdown ens18 && sudo ifup ens18
```

---

## OpenSSH serveri paigaldamine

SSH võimaldab turvalist kaugühendust serverisse.

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

---

## Kontrollimine

```bash
hostnamectl                  # hostname kontroll
ip a                         # IP-aadressi kontroll
systemctl status ssh         # SSH teenuse staatus
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Võrk ei tule üles | Kontrolli liidese nime käsuga `ip link` |
| SSH ei tööta | `journalctl -u ssh` näitab logisid |
| DNS ei tööta | Kontrolli `/etc/resolv.conf` sisu |

---

## Levinud vead

- **Vale liidese nimi** – `interfaces` failis peab olema õige nimi (nt `ens18`, `eth0`)
- **Firewall blokeerib port 22** – kui kasutad UFW-d, lisa reegel `sudo ufw allow ssh`
- **Networking teenus ei käivitu** – kontrolli konfiguratsiooni süntaksit
