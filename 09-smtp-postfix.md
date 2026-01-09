# SMTP Server (Postfix)

> Postfix on populaarne ja turvaline meiliserver. See juhend näitab põhiseadistust lokaalse meili jaoks.

[Tagasi README](README.md) · [← Eelmine](08-nginx.md) · [Järgmine →](10-tftp.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- DNS on konfigureeritud (MX kirje)
- Kasutajal on sudo õigused

---

## DNS kirjete lisamine

Lisa DNS tsooni:
```text
mail    IN      A       10.0.80.2
@       IN      MX  10  mail.plaas.lan.
```

> **NB!** Ära unusta Serial numbrit tõsta.

---

## Paigaldamine

```bash
sudo apt update
sudo apt install postfix -y
```

Installimisel vali:
- **Internet Site** – meili saatmine ja vastuvõtmine otse

---

## Konfigureerimine

```bash
sudo dpkg-reconfigure postfix
```

Seaded:
- **System mail name:** `mail.plaas.lan`
- **Root and postmaster recipient:** (jäta tühjaks või sisesta kasutaja)
- **Other destinations:** `plaas.lan, localhost`
- **Local networks:** `127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 10.0.80.0/24`

```bash
sudo systemctl reload postfix
```

---

## Meilitööriistade paigaldamine

```bash
sudo apt install mailutils mutt -y
```

---

## Testimine

Kirja saatmine:
```bash
echo "Tere! See on testmeil." | mail -s "Testteade" kasutaja@plaas.lan
```

Kirjade vaatamine:
```bash
mutt
```

või
```bash
mail
```

---

## Kontrollimine

```bash
systemctl status postfix
sudo tail -f /var/log/mail.log
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Meil ei lähe | `tail -f /var/log/mail.log` |
| MX puudu | Lisa DNS kirje ja tõsta Serial |
| Relay keelatud | Kontrolli `mynetworks` seadet |

---

## Levinud vead

- **MX kirje puudu või vale** – meili ei suunata õigesse serverisse
- **Hostname ei vasta** – `myhostname` peab vastama DNS-ile
- **Tulemüür blokeerib** – port 25 peab olema lubatud
