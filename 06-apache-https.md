# Apache2 HTTPS + vhostid
[Tagasi README](README.md) · [← Eelmine](05-ufw.md) · [Järgmine →](07-almalinux-staatiline-ip.md)

## Sertid
```bash
sudo apt install apache2
mkdir ~/ssl && cd ~/ssl
openssl req -nodes -new -keyout www.plaas.lan.key -newkey rsa:2048 > www.plaas.lan.csr
openssl x509 -req -days 3650 -in www.plaas.lan.csr -signkey www.plaas.lan.key -out www.plaas.lan.crt
openssl req -nodes -new -keyout sales.plaas.lan.key -newkey rsa:2048 > sales.plaas.lan.csr
openssl x509 -req -days 3650 -in sales.plaas.lan.csr -signkey sales.plaas.lan.key -out sales.plaas.lan.crt
sudo mkdir /etc/apache2/ssl
sudo cp ~/ssl/* /etc/apache2/ssl
```

## Vhostid
```bash
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/www-ssl.conf
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/sales-ssl.conf

# www-ssl.conf
# ServerName www.plaas.lan
# SSLCertificateFile /etc/apache2/ssl/www.plaas.lan.crt
# SSLCertificateKeyFile /etc/apache2/ssl/www.plaas.lan.key

# sales-ssl.conf
# ServerName sales.plaas.lan
# SSLCertificateFile /etc/apache2/ssl/sales.plaas.lan.crt
# SSLCertificateKeyFile /etc/apache2/ssl/sales.plaas.lan.key

sudo a2enmod ssl
sudo a2ensite www-ssl
sudo a2ensite sales-ssl
sudo systemctl reload apache2
```

## DNS A kirjed
```bash
# tõsta serial
# lisa:
# www.plaas.lan. IN A 10.0.80.2
# sales.plaas.lan. IN A 10.0.80.2
sudo systemctl restart bind9
```

## Dokumendikaustad
```bash
sudo mkdir -p /var/www/www /var/www/sales
printf '<h1>www</h1>' | sudo tee /var/www/www/index.html
printf '<h1>sales</h1>' | sudo tee /var/www/sales/index.html
sudo systemctl reload apache2
```

## HTTPS suunamised (:80 → :443)
```apache
<VirtualHost *:80>
  ServerName sales.plaas.lan
  Redirect / https://sales.plaas.lan/
</VirtualHost>
<VirtualHost *:80>
  ServerName www.plaas.lan
  Redirect / https://www.plaas.lan/
</VirtualHost>
```

## Vigade leidmine ja parandamine
- Sertide teed: kontrolli `SSLCertificateFile/KeyFile`
- Vhostide enabled: `a2ensite` + `systemctl reload apache2`

## Levinud vead
- DNS serial unustatakse tõsta
