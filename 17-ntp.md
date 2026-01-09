# NTP Ajasünkroniseerimine

> NTP (Network Time Protocol) tagab, et kõik võrgus olevad seadmed näitavad õiget aega. See on oluline logide, sertifikaatide ja autentimise jaoks.

[Tagasi README](README.md) · [← Eelmine](16-iscsi.md) · [Järgmine →](18-zabbix.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Port 123/UDP on vaba

---

## NTP Server (systemd-timesyncd)

### Paigaldamine

```bash
sudo apt install -y systemd-timesyncd
```

### Ajavööndi seadistamine

```bash
sudo timedatectl set-timezone Europe/Tallinn
```

### Konfigureerimine

```bash
sudo nano /etc/systemd/timesyncd.conf
```

Sisu:
```text
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org
FallbackNTP=2.pool.ntp.org 3.pool.ntp.org
```

### Teenuse käivitamine

```bash
sudo systemctl enable --now systemd-timesyncd
```

### Kontrollimine

```bash
timedatectl status
timedatectl show-timesync --all
```

### Tulemüür

```bash
sudo ufw allow 123/udp
```

---

## NTP Klient (Debian/Ubuntu)

```bash
sudo apt install -y systemd-timesyncd
sudo timedatectl set-timezone Europe/Tallinn
```

```bash
sudo nano /etc/systemd/timesyncd.conf
```

```text
[Time]
NTP=server-ip
```

```bash
sudo systemctl restart systemd-timesyncd
```

---

## NTP Klient (Windows)

### PowerShell

```powershell
# Ajavööndi seadmine
tzutil /s "FLE Standard Time"

# NTP serveri seadmine
w32tm /config /manualpeerlist:"server-ip" /syncfromflags:manual /update

# Teenuse taaskäivitamine
net stop w32time
net start w32time

# Sünkroniseerimine
w32tm /resync

# Staatuse kontroll
w32tm /query /status
w32tm /query /configuration
```

### GUI kaudu

1. **Settings** → **Time & Language** → **Date & time**
2. **Additional date, time & regional settings**
3. **Set the time and date** → **Internet Time** → **Change settings**
4. Sisesta serveri IP ja vajuta **Update now**

---

## Chrony (alternatiiv)

```bash
sudo apt install chrony -y
sudo nano /etc/chrony/chrony.conf
```

Lisa:
```text
server server-ip iburst
```

```bash
sudo systemctl restart chrony
chronyc tracking
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Aeg ei sünkroniseeru | Kontrolli võrguühendust serveriga |
| Vale ajavöönd | `timedatectl set-timezone` |
| Timeout | Kontrolli port 123/UDP tulemüüris |

---

## Levinud vead

- **UDP/123 blokeeritud** – NTP kasutab porti 123/UDP
- **Vale ajavöönd** – kontrolli `timedatectl status`
- **Tulemüür mõlemal pool** – nii serveril kui kliendil peab port olema avatud
