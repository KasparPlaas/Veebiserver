# NFS Failijagamine

> NFS (Network File System) võimaldab jagada kaustasid Unix/Linux süsteemide vahel läbipaistvalt, nagu oleksid need lokaalsed.

[Tagasi README](README.md) · [← Eelmine](13-samba.md) · [Järgmine →](15-squid.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Klient ja server on samas võrgus

---

## NFS serveri paigaldamine

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

---

## Jagatud kausta loomine

```bash
sudo mkdir -p /srv/nfs
sudo chmod 777 /srv/nfs
```

---

## Eksportimise seadistamine

```bash
sudo nano /etc/exports
```

Lisa rida:
```text
/srv/nfs    10.0.80.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Parameetrite selgitus:
- `rw` – lugemis- ja kirjutamisõigus
- `sync` – kirjutamine kohe kettale
- `no_subtree_check` – kiirem, vähem kontrollimist
- `no_root_squash` – root kasutaja säilitab õigused

Eksportide rakendamine:
```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```

---

## Tulemüür

```bash
sudo ufw allow from 10.0.80.0/24 to any port nfs
sudo ufw allow 111
sudo ufw allow 2049
```

---

## NFS klient

### Debian
```bash
sudo apt install nfs-common -y
```

### AlmaLinux
```bash
sudo dnf install nfs-utils -y
```

### Haakimine (mount)

```bash
sudo mkdir -p /mnt/nfs
sudo mount -t nfs server-ip:/srv/nfs /mnt/nfs
```

Kontrollimine:
```bash
df -h
```

### Püsiühendus `/etc/fstab`

```text
server-ip:/srv/nfs    /mnt/nfs    nfs    defaults    0    0
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Ei saa ühendada | `exportfs -v` kontrolli eksporte |
| Permission denied | Kontrolli kausta õigusi |
| Timeout | Kontrolli tulemüüri (port 2049) |

---

## Levinud vead

- **Port 2049 blokeeritud** – NFS kasutab porti 2049
- **Exports failis viga** – kontrolli süntaksit ja IP vahemikku
- **Õigused valed** – kasuta `no_root_squash` kui vaja root ligipääsu
