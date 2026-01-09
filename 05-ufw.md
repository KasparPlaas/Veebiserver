# UFW tulemüür
[Tagasi README](README.md) · [← Eelmine](04-dhcp.md) · [Järgmine →](06-apache-https.md)

## Kiirseadistus
```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw allow bind9
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 67
sudo ufw enable
```

## Vigade leidmine ja parandamine
- Port puudu: lisa `ufw allow <port>`

## Levinud vead
- UFW lubamata (`enable`) pärast reegleid
