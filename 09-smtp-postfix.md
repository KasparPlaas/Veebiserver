# SMTP (Postfix)
[Tagasi README](README.md) · [← Eelmine](08-nginx.md) · [Järgmine →](10-tftp.md)

## DNS kirjed
```text
mail.plaas.lan. IN A 10.0.80.2
plaas.lan. 3d IN MX 10 mail.plaas.lan.
```

## Install ja seadistus
```bash
sudo apt install postfix
# vali "Internet Site"
dpkg-reconfigure postfix
# System mail name: mail.plaas.lan
# Muud seaded vastavalt sinu sisule
sudo systemctl reload postfix
sudo apt install mailutils mutt
```

## Kiirtest
```bash
# kiri
echo "Tere" | mail -s "Test" kasutaja@minu.lan
# vaata
mutt
```

## Vigade leidmine ja parandamine
- `mail.log` vaata: `tail -f /var/log/mail.log`

## Levinud vead
- MX puudu või vale
