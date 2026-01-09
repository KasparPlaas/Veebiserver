# Apache2 HTTPS + Virtual Hosts

> Apache2 on üks populaarsemaid veebiservereid. See juhend näitab, kuidas seadistada HTTPS ja virtuaalhostid.

[Tagasi README](README.md) · [← Eelmine](05-ufw.md) · [Järgmine →](07-almalinux-staatiline-ip.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- DNS on konfigureeritud (domeenid lahenduvad)
- Kasutajal on sudo õigused

---

## Apache2 paigaldamine

```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl enable --now apache2
```

---

## SSL sertifikaatide genereerimine

### Sertifikaatide kaust

```bash
mkdir ~/ssl && cd ~/ssl
sudo mkdir -p /etc/apache2/ssl
```

### Sertifikaatide loomine

```bash
# www.plaas.lan
openssl req -nodes -new -keyout www.plaas.lan.key -newkey rsa:2048 -out www.plaas.lan.csr
openssl x509 -req -days 3650 -in www.plaas.lan.csr -signkey www.plaas.lan.key -out www.plaas.lan.crt

# sales.plaas.lan
openssl req -nodes -new -keyout sales.plaas.lan.key -newkey rsa:2048 -out sales.plaas.lan.csr
openssl x509 -req -days 3650 -in sales.plaas.lan.csr -signkey sales.plaas.lan.key -out sales.plaas.lan.crt

# Kopeeri serdid Apache kausta
sudo cp ~/ssl/* /etc/apache2/ssl/
```

---

## Virtual Hostide seadistamine

### Konfiguratsioonifailid

```bash
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/www-ssl.conf
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/sales-ssl.conf
```

### www-ssl.conf näidis

```bash
sudo nano /etc/apache2/sites-available/www-ssl.conf
```

```apache
<VirtualHost *:443>
    ServerName www.plaas.lan
    DocumentRoot /var/www/www
    
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/www.plaas.lan.crt
    SSLCertificateKeyFile /etc/apache2/ssl/www.plaas.lan.key
</VirtualHost>
```

### sales-ssl.conf näidis

```bash
sudo nano /etc/apache2/sites-available/sales-ssl.conf
```

```apache
<VirtualHost *:443>
    ServerName sales.plaas.lan
    DocumentRoot /var/www/sales
    
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/sales.plaas.lan.crt
    SSLCertificateKeyFile /etc/apache2/ssl/sales.plaas.lan.key
</VirtualHost>
```

---

## Saitide aktiveerimine

```bash
sudo a2enmod ssl
sudo a2ensite www-ssl
sudo a2ensite sales-ssl
sudo systemctl reload apache2
```

---

## Dokumendikaustad

```bash
sudo mkdir -p /var/www/www /var/www/sales
echo '<h1>www.plaas.lan</h1>' | sudo tee /var/www/www/index.html
echo '<h1>sales.plaas.lan</h1>' | sudo tee /var/www/sales/index.html
```

---

## DNS A-kirjed

Lisa DNS serverisse:
```text
www     IN      A       10.0.80.2
sales   IN      A       10.0.80.2
```

> **NB!** Ära unusta Serial numbrit tõsta ja BIND9 restartida.

---

## HTTP → HTTPS suunamine

```bash
sudo nano /etc/apache2/sites-available/redirect.conf
```

```apache
<VirtualHost *:80>
    ServerName www.plaas.lan
    Redirect permanent / https://www.plaas.lan/
</VirtualHost>

<VirtualHost *:80>
    ServerName sales.plaas.lan
    Redirect permanent / https://sales.plaas.lan/
</VirtualHost>
```

```bash
sudo a2ensite redirect
sudo systemctl reload apache2
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Sert ei tööta | Kontrolli `SSLCertificateFile` ja `KeyFile` teid |
| Sait ei ilmu | `a2ensite` ja `systemctl reload apache2` |
| DNS ei lahenda | Tõsta Serial ja tee `systemctl restart bind9` |

---

## Levinud vead

- **DNS Serial muutmata** – muudatused ei rakendu
- **Sertifikaadi tee vale** – kontrolli failide asukohta
- **Moodul lubamata** – `a2enmod ssl` unustatud
