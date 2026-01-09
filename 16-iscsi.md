# iSCSI Storage

> iSCSI võimaldab pakkuda blokkseadmeid üle võrgu, mis tunduvad klientidele nagu lokaalsed kettad. Seda kasutatakse SAN (Storage Area Network) lahendustes.

[Tagasi README](README.md) · [← Eelmine](15-squid.md) · [Järgmine →](17-ntp.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Piisavalt kettaruumi virtuaalse ketta jaoks

---

## iSCSI Target (Server)

### Paigaldamine

```bash
sudo apt update
sudo apt install -y targetcli-fb
```

### Virtuaalse ketta loomine

```bash
sudo mkdir -p /srv/iscsi_disks
sudo dd if=/dev/zero of=/srv/iscsi_disks/disk01.img bs=1M count=1024
```

See loob 1GB suuruse kettapildi.

### Targetcli seadistamine

```bash
sudo targetcli
```

Targetcli sees:
```text
# Backstorage loomine
/backstores/fileio create disk01 /srv/iscsi_disks/disk01.img

# iSCSI target loomine
/iscsi create iqn.2025-01.plaas.lan:target01

# LUN lisamine
/iscsi/iqn.2025-01.plaas.lan:target01/tpg1/luns create /backstores/fileio/disk01

# Portali loomine
/iscsi/iqn.2025-01.plaas.lan:target01/tpg1/portals create 0.0.0.0 3260

# ACL (klientide lubamine)
/iscsi/iqn.2025-01.plaas.lan:target01/tpg1/acls create iqn.2025-01.plaas.lan:debian-client

# Autentimise keelamine (testimiseks)
/iscsi/iqn.2025-01.plaas.lan:target01/tpg1 set attribute authentication=0

# Salvestamine ja väljumine
saveconfig
exit
```

### Teenuse käivitamine

```bash
sudo systemctl enable --now rtslib-fb-targetctl
```

### Tulemüür

```bash
sudo ufw allow 3260/tcp
```

---

## iSCSI Initiator (Debian klient)

### Paigaldamine

```bash
sudo apt install -y open-iscsi
```

### Initiator nime määramine

```bash
sudo nano /etc/iscsi/initiatorname.iscsi
```

```text
InitiatorName=iqn.2025-01.plaas.lan:debian-client
```

### Teenuse käivitamine

```bash
sudo systemctl enable --now iscsid
```

### Target avastamine ja ühendamine

```bash
# Targetite avastamine
sudo iscsiadm -m discovery -t sendtargets -p server-ip

# Ühendamine
sudo iscsiadm -m node -T iqn.2025-01.plaas.lan:target01 -p server-ip --login

# Kontrollimine
lsblk
```

### Automaatne ühendamine

```bash
sudo iscsiadm -m node -T iqn.2025-01.plaas.lan:target01 --op update -n node.startup -v automatic
```

---

## iSCSI Initiator (Windows)

### PowerShell

```powershell
# Portali lisamine
iscsicli QAddTargetPortal server-ip 3260

# Targetite vaatamine
iscsicli ListTargets

# Ühendamine
iscsicli QLoginTarget iqn.2025-01.plaas.lan:target01

# Sessiooni kontroll
iscsicli SessionList

# Püsiühendus
iscsicli PersistentLoginTarget iqn.2025-01.plaas.lan:target01 T * * * * * * * * * * * * * * * 0
```

### GUI kaudu

1. Käivita `iscsicpl.exe` (iSCSI Initiator)
2. **Discovery** → **Discover Portal** → sisesta serveri IP
3. **Targets** → vali target → **Connect**
4. **Disk Management** → initsialiseeri ja formaadi uus ketas

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Port 3260 blokeeritud | Kontrolli tulemüüri |
| `lsblk` ei näita ketast | Kontrolli, kas sessioon on loodud |
| ACL viga | IQN peab täpselt kattuma |

---

## Levinud vead

- **Vale IQN** – initiator ja target ACL peavad täpselt kattuma
- **Tulemüür** – port 3260/TCP peab olema avatud
- **Sessioon puudu** – `iscsiadm --login` unustatud
