# NGINX Virtual Hosts

> NGINX on kiire ja efektiivne veebiserver ning reverse proxy. See juhend näitab virtual hostide seadistamist.

[Tagasi README](README.md) · [← Eelmine](07-almalinux-staatiline-ip.md) · [Järgmine →](09-smtp-postfix.md)

---

## Eeldused

- NGINX on paigaldatud
- DNS on konfigureeritud
- Kasutajal on sudo õigused

---

## Virtual Host konfiguratsioon

### Konfiguratsioonifaili loomine

```bash
sudo nano /etc/nginx/conf.d/example.conf
```

Näidiskonfiguratsioon:
```nginx
server {
    listen       80;
    listen       [::]:80;
    server_name  example.com www.example.com;
    
    root         /usr/share/nginx/example;
    index        index.html index.htm;
    
    access_log   /var/log/nginx/example.com_access.log;
    error_log    /var/log/nginx/example.com_error.log;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Saidi kausta loomine

```bash
sudo mkdir -p /usr/share/nginx/example
echo '<h1>Tere tulemast example.com</h1>' | sudo tee /usr/share/nginx/example/index.html
```

---

## Konfiguratsiooni kontrollimine

```bash
sudo nginx -t                    # süntaksi kontroll
sudo systemctl reload nginx      # muudatuste rakendamine
```

---

## HTTPS tugi (self-signed)

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/ssl/example.key \
    -out /etc/nginx/ssl/example.crt
```

HTTPS konfiguratsioon:
```nginx
server {
    listen       443 ssl;
    server_name  example.com;
    
    ssl_certificate     /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;
    
    root         /usr/share/nginx/example;
    index        index.html;
}
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Konfiviga | `nginx -t` näitab täpset viga |
| Muudatused ei rakendu | `systemctl reload nginx` |
| 403 Forbidden | Kontrolli kausta õigusi |

---

## Levinud vead

- **Vale root kaust** – kontrolli, et kaust eksisteerib
- **Süntaksi viga** – puuduvad semikoolonid või sulud
- **SELinux (AlmaLinux)** – võib blokeerida ligipääsu kaustadele
