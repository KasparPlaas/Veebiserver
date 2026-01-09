# Ubuntu Basic Configuration

> Ubuntu Server on populaarne Linux distributsioon, mis kasutab Netplan võrgukonfiguratsiooni. See juhend aitab seadistada baaskonfiguratsiooni.

[Tagasi README](README.md) · [← Eelmine](01-debian-alustamine.md) · [Järgmine →](03-bind9.md)

---

## Eeldused

- Ubuntu Server 22.04 LTS või uuem on paigaldatud
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

## Staatiline IP-aadress (Netplan)

Ubuntu kasutab Netplan süsteemi võrgu konfigureerimiseks.

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

Näidiskonfiguratsioon:
```yaml
network:
  version: 2
  ethernets:
    ens18:
      addresses:
        - 10.0.80.4/24
      routes:
        - to: default
          via: 10.0.80.1
      nameservers:
        addresses:
          - 8.8.8.8
```

> **NB!** YAML failis kasuta ainult tühikuid (2 tühikut taandeks), mitte tabulaatoreid.

Muudatuste rakendamine:
```bash
sudo netplan apply
```

Konfiguratsiooni testimine (taastub automaatselt vea korral):
```bash
sudo netplan try
```

---

## OpenSSH serveri paigaldamine

SSH võimaldab turvalist kaugühendust serverisse.

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
```

UFW tulemüüri seadistamine:
```bash
sudo ufw allow ssh
sudo ufw enable
```

---

## Kontrollimine

```bash
hostnamectl                  # hostname kontroll
ip a                         # IP-aadressi kontroll
systemctl status ssh         # SSH teenuse staatus
sudo ufw status              # tulemüüri staatus
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Netplan viga | `sudo netplan try` testimiseks |
| SSH ei tööta | `sudo ufw status` kontrolli reegleid |
| Võrk puudub | `ip link` kontrolli liidese nime |

---

## Levinud vead

- **YAML taande viga** – kasuta tühikuid (2 tk), mitte tabulaatoreid
- **UFW pole lubatud** – `sudo ufw enable` käivitab tulemüüri
- **gateway4 on aegunud** – uuemates versioonides kasuta `routes` sektsiooni
- **Vale liidese nimi** – kontrolli `ip link` käsuga õiget nime
