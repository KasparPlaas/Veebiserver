# AlmaLinux Basic Configuration

> AlmaLinux on RHEL-põhine ettevõtteklass Linux distributsioon. See juhend aitab seadistada baaskonfiguratsiooni serveri jaoks.

<p align="center">
  <a href="README.md"><img src="https://img.shields.io/badge/Tagasi-README-blue?style=for-the-badge" alt="Tagasi"></a>
  <a href="01-debian-alustamine.md"><img src="https://img.shields.io/badge/Järgmine-Debian_Basic_Config-A81D33?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- AlmaLinux 9.x on paigaldatud
- Kasutajal on sudo õigused
- Võrguühendus on olemas (kas DHCP või ajutine)

---

## Hostname seadistamine

Hostname määrab masina identiteedi võrgus.

```bash
sudo hostnamectl set-hostname srv
sudo nano /etc/hosts
```

Lisa või muuda rida:
```text
127.0.1.1   srv
```

---

## Staatiline IP-aadress

### Variant 1: nmtui (graafiline)

```bash
sudo nmtui
```

Tegevused:
1. **Edit a connection** → vali võrguliides (nt `ens18`)
2. **IPv4 CONFIGURATION** → muuda `<Automatic>` → `<Manual>`
3. Lisa **Addresses** (nt `10.0.80.3/24`)
4. Lisa **Gateway** (nt `10.0.80.1`)
5. Lisa **DNS servers** (nt `8.8.8.8`)
6. Salvesta ja välju
7. **Activate a connection** → deaktiveeri ja aktiveeri uuesti

### Variant 2: nmcli (käsurida)

```bash
sudo nmcli con mod ens18 ipv4.addresses 10.0.80.3/24
sudo nmcli con mod ens18 ipv4.gateway 10.0.80.1
sudo nmcli con mod ens18 ipv4.dns "8.8.8.8"
sudo nmcli con mod ens18 ipv4.method manual
sudo nmcli con up ens18
```

---

## OpenSSH serveri paigaldamine

SSH võimaldab turvalist kaugühendust serverisse.

```bash
sudo dnf install openssh-server -y
sudo systemctl enable --now sshd
```

Firewall reeglite lisamine:
```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

---

## Kontrollimine

```bash
hostnamectl                  # hostname kontroll
ip a                         # IP-aadressi kontroll
systemctl status sshd        # SSH teenuse staatus
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Võrk ei tule üles | `nmcli con show` ja kontrolli seadeid |
| SSH ei tööta | `firewall-cmd --list-services` |
| Ühendus katkeb | Kontrolli gateway ja DNS seadeid |

---

## Levinud vead

- **Firewalld reeglid lisamata** – SSH port 22 peab olema lubatud
- **nmtui muudatusi ei aktiveerita** – peale salvestamist aktiveeri ühendus uuesti
- **Vale liidese nimi** – kontrolli `ip link` käsuga õiget nime
