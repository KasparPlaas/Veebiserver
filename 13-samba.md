# SAMBA Failijagamine

> SAMBA võimaldab jagada faile ja kaustasid Linuxi ja Windowsi masinate vahel, kasutades SMB/CIFS protokolli.

<p align="center">
  <a href="12-sftp.md"><img src="https://img.shields.io/badge/Eelmine-SFTP-231F20?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="14-load-balancer.md"><img src="https://img.shields.io/badge/Järgmine-Load_Balancer-FF6B6B?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Võrguühendus on olemas

---

## Paigaldamine

```bash
sudo apt update
sudo apt install samba -y
```

---

## Jagatud kausta loomine

```bash
sudo mkdir -p /srv/jagatud
sudo chmod 777 /srv/jagatud
```

---

## Konfigureerimine

```bash
sudo nano /etc/samba/smb.conf
```

Lisa faili lõppu:
```text
[jagatud]
    path = /srv/jagatud
    browseable = yes
    guest ok = yes
    read only = no
    public = yes
    writable = yes
    create mask = 0664
    directory mask = 0775
```

Teenuse taaskäivitamine:
```bash
sudo systemctl restart smbd nmbd
sudo systemctl enable smbd nmbd
```

---

## Tulemüür

```bash
sudo ufw allow samba
```

või käsitsi:
```bash
sudo ufw allow 139/tcp
sudo ufw allow 445/tcp
```

---

## Kliendi ühendamine

### Linux

```bash
sudo apt install smbclient cifs-utils -y
```

Kausta vaatamine:
```bash
smbclient -L //server-ip -N
```

Püsiühendus:
```bash
sudo mkdir -p /mnt/vorgukaust
sudo mount -t cifs //server-ip/jagatud /mnt/vorgukaust -o guest
```

Püsiühendus `/etc/fstab` kaudu:
```text
//server-ip/jagatud  /mnt/vorgukaust  cifs  guest,uid=1000  0  0
```

### Windows

1. Ava File Explorer
2. Vajuta **Map network drive**
3. Sisesta: `\\server-ip\jagatud`
4. Märgi "Connect using different credentials" kui vaja

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Ühendus ebaõnnestub | Kontrolli tulemüüri reegleid |
| Ei saa kirjutada | Kontrolli kausta õigusi |
| Windows ei näe | Kontrolli SMB versiooni |

---

## Levinud vead

- **Tulemüür blokeerib** – pordid 139 ja 445 peavad olema avatud
- **Kausta õigused** – Linux pool peab võimaldama kirjutamist
- **Windows backslash** – Windowsis kasuta `\\`, mitte `/`
