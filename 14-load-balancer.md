# Koormusjaoturid (Load Balancer)

> Koormusjaotur jaotab võrguliiklust mitme serveri vahel, tagades kõrge käideldavuse ja parema jõudluse. See juhend käsitleb NGINX, HAProxy ja Apache koormusjaoturite seadistamist.

<p align="center">
  <a href="13-samba.md"><img src="https://img.shields.io/badge/Eelmine-SAMBA-007ACC?style=for-the-badge" alt="Eelmine"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/README-blue?style=for-the-badge" alt="README"></a>
  <a href="15-nfs.md"><img src="https://img.shields.io/badge/Järgmine-NFS-6DB33F?style=for-the-badge" alt="Järgmine"></a>
</p>

---

## Eeldused

- Debian 12 server on seadistatud (koormusjaotur)
- Vähemalt 2 backend serverit (veebiserverid)
- Kõik serverid on samas võrgus
- Kasutajal on sudo õigused

---

## Arhitektuur

```
                    ┌─────────────────┐
                    │   Kliendid      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Load Balancer   │
                    │  (10.0.80.10)   │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
      ┌───────────┐  ┌───────────┐  ┌───────────┐
      │ Backend 1 │  │ Backend 2 │  │ Backend 3 │
      │ 10.0.80.11│  │ 10.0.80.12│  │ 10.0.80.13│
      └───────────┘  └───────────┘  └───────────┘
```

---

# NGINX Load Balancer

## Paigaldamine

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable --now nginx
```

---

## Põhikonfiguratsioon

```bash
sudo nano /etc/nginx/conf.d/load-balancer.conf
```

Sisu:
```nginx
upstream backend_servers {
    # Koormuse jaotamise meetod (vaikimisi round-robin)
    # least_conn;      # Vähima ühenduste arvuga serverile
    # ip_hash;         # Sama IP alati samale serverile
    
    server 10.0.80.11:80 weight=3;    # Suurem kaal = rohkem liiklust
    server 10.0.80.12:80 weight=2;
    server 10.0.80.13:80 weight=1;
    
    # Tervisekontroll
    # server 10.0.80.11:80 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name loadbalancer.plaas.lan;
    
    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeout seaded
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

Vaikimisi saidi keelamine ja testi:
```bash
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

## NGINX + SSL Termination

SSL termineerimine tähendab, et HTTPS ühendused lõpetatakse koormusjaoturis ja backend serveritega suheldakse HTTP kaudu.

### Sertifikaadi loomine

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/nginx/ssl/loadbalancer.key \
    -out /etc/nginx/ssl/loadbalancer.crt \
    -subj "/CN=loadbalancer.plaas.lan"
```

### HTTPS konfiguratsioon

```bash
sudo nano /etc/nginx/conf.d/load-balancer-ssl.conf
```

```nginx
upstream backend_servers {
    least_conn;
    server 10.0.80.11:80;
    server 10.0.80.12:80;
    server 10.0.80.13:80;
}

# HTTP -> HTTPS suunamine
server {
    listen 80;
    server_name loadbalancer.plaas.lan;
    return 301 https://$host$request_uri;
}

# HTTPS server
server {
    listen 443 ssl;
    server_name loadbalancer.plaas.lan;
    
    ssl_certificate /etc/nginx/ssl/loadbalancer.crt;
    ssl_certificate_key /etc/nginx/ssl/loadbalancer.key;
    
    # SSL turvaseaded
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    
    location / {
        proxy_pass http://backend_servers;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# HAProxy Load Balancer

## Paigaldamine

```bash
sudo apt update
sudo apt install haproxy -y
sudo systemctl enable --now haproxy
```

---

## Põhikonfiguratsioon

Varukoopia:
```bash
sudo cp /etc/haproxy/haproxy.cfg{,.bak}
```

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Sisu:
```haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http

# Statistika leht
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
    stats admin if LOCALHOST

# Frontend - kuulab sissetulevaid ühendusi
frontend http_front
    bind *:80
    default_backend http_back
    
    # ACL näited
    # acl is_api path_beg /api
    # use_backend api_back if is_api

# Backend - serverite grupp
backend http_back
    balance roundrobin
    option httpchk GET /
    http-check expect status 200
    
    server web1 10.0.80.11:80 check
    server web2 10.0.80.12:80 check
    server web3 10.0.80.13:80 check backup
```

Kontrolli ja taaskäivita:
```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```

---

## HAProxy + SSL Termination

### Sertifikaadi ettevalmistamine

HAProxy vajab sertifikaati ja võtit ühes failis (PEM formaat):

```bash
sudo mkdir -p /etc/haproxy/certs

# Loo self-signed sertifikaat
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/haproxy/certs/loadbalancer.key \
    -out /etc/haproxy/certs/loadbalancer.crt \
    -subj "/CN=loadbalancer.plaas.lan"

# Kombineeri üheks failiks
sudo cat /etc/haproxy/certs/loadbalancer.crt /etc/haproxy/certs/loadbalancer.key | sudo tee /etc/haproxy/certs/loadbalancer.pem
sudo chmod 600 /etc/haproxy/certs/loadbalancer.pem
```

### HTTPS konfiguratsioon

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Lisa/muuda:
```haproxy
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
    
    # SSL seaded
    ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
    ssl-default-bind-options ssl-min-ver TLSv1.2

defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    timeout connect 5000
    timeout client  50000
    timeout server  50000

# Statistika
listen stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s

# HTTPS Frontend
frontend https_front
    bind *:443 ssl crt /etc/haproxy/certs/loadbalancer.pem
    default_backend http_back
    
    # Lisa X-Forwarded headerid
    http-request set-header X-Forwarded-Proto https
    http-request set-header X-Forwarded-Port 443

# HTTP -> HTTPS suunamine
frontend http_front
    bind *:80
    redirect scheme https code 301

# Backend
backend http_back
    balance roundrobin
    option httpchk GET /
    
    server web1 10.0.80.11:80 check
    server web2 10.0.80.12:80 check
    server web3 10.0.80.13:80 check
```

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
sudo systemctl restart haproxy
```

---

## HAProxy statistika

Ava brauseris: `http://loadbalancer-ip:8404/stats`

---

# Apache Load Balancer (mod_proxy)

## Paigaldamine

```bash
sudo apt update
sudo apt install apache2 -y
sudo a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests headers
sudo systemctl enable --now apache2
```

---

## Põhikonfiguratsioon

```bash
sudo nano /etc/apache2/sites-available/load-balancer.conf
```

Sisu:
```apache
<VirtualHost *:80>
    ServerName loadbalancer.plaas.lan
    
    # Proxy seaded
    ProxyPreserveHost On
    ProxyRequests Off
    
    # Tervisekontroll
    ProxyPass /balancer-manager !
    
    # Backend serverite grupp
    <Proxy "balancer://mycluster">
        BalancerMember "http://10.0.80.11:80" route=web1 loadfactor=3
        BalancerMember "http://10.0.80.12:80" route=web2 loadfactor=2
        BalancerMember "http://10.0.80.13:80" route=web3 loadfactor=1
        
        # Koormuse jaotamise meetod
        ProxySet lbmethod=byrequests
        # Alternatiivsed meetodid:
        # ProxySet lbmethod=bytraffic
        # ProxySet lbmethod=bybusyness
    </Proxy>
    
    # Proxy päringud backend serveritele
    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
    
    # Balancer manager (ainult lokaalvõrgust)
    <Location "/balancer-manager">
        SetHandler balancer-manager
        Require ip 10.0.80.0/24
    </Location>
    
    # Headerid
    RequestHeader set X-Forwarded-Proto "http"
    
    ErrorLog ${APACHE_LOG_DIR}/lb_error.log
    CustomLog ${APACHE_LOG_DIR}/lb_access.log combined
</VirtualHost>
```

Aktiveeri ja taaskäivita:
```bash
sudo a2ensite load-balancer.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## Apache + SSL Termination

### Sertifikaadi loomine

```bash
sudo mkdir -p /etc/apache2/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/apache2/ssl/loadbalancer.key \
    -out /etc/apache2/ssl/loadbalancer.crt \
    -subj "/CN=loadbalancer.plaas.lan"
```

### HTTPS konfiguratsioon

```bash
sudo a2enmod ssl
sudo nano /etc/apache2/sites-available/load-balancer-ssl.conf
```

```apache
<VirtualHost *:80>
    ServerName loadbalancer.plaas.lan
    Redirect permanent / https://loadbalancer.plaas.lan/
</VirtualHost>

<VirtualHost *:443>
    ServerName loadbalancer.plaas.lan
    
    # SSL seaded
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/loadbalancer.crt
    SSLCertificateKeyFile /etc/apache2/ssl/loadbalancer.key
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    
    # Proxy seaded
    ProxyPreserveHost On
    ProxyRequests Off
    
    <Proxy "balancer://mycluster">
        BalancerMember "http://10.0.80.11:80" route=web1
        BalancerMember "http://10.0.80.12:80" route=web2
        BalancerMember "http://10.0.80.13:80" route=web3
        ProxySet lbmethod=byrequests
    </Proxy>
    
    ProxyPass "/" "balancer://mycluster/"
    ProxyPassReverse "/" "balancer://mycluster/"
    
    # HTTPS header backend serveritele
    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"
    
    <Location "/balancer-manager">
        SetHandler balancer-manager
        Require ip 10.0.80.0/24
    </Location>
    
    ErrorLog ${APACHE_LOG_DIR}/lb_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/lb_ssl_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite load-balancer-ssl.conf
sudo a2dissite load-balancer.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## Apache Balancer Manager

Ava brauseris: `http://loadbalancer-ip/balancer-manager`

Võimaldab:
- Näha serverite staatust
- Muuta serverite kaalu
- Keelata/lubada servereid

---

## Tulemüür

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8404/tcp    # HAProxy stats (vajadusel)
```

---

## Koormuse jaotamise meetodid

| Meetod | NGINX | HAProxy | Apache | Kirjeldus |
|--------|-------|---------|--------|-----------|
| Round Robin | vaikimisi | `roundrobin` | `byrequests` | Järjestikku igale serverile |
| Least Connections | `least_conn` | `leastconn` | `bybusyness` | Vähima koormusega serverile |
| IP Hash | `ip_hash` | `source` | - | Sama klient alati samale serverile |
| Weighted | `weight=N` | `weight N` | `loadfactor=N` | Kaalutud jaotus |

---

## Kontrollimine

### Backend serverite test

```bash
# Testi igat serverit eraldi
curl -I http://10.0.80.11
curl -I http://10.0.80.12
curl -I http://10.0.80.13
```

### Koormusjaoturi test

```bash
# Mitu päringut, jälgi vastuseid
for i in {1..10}; do curl -s http://loadbalancer-ip | grep -o "Server[0-9]"; done

# HTTPS test
curl -k https://loadbalancer-ip
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| 502 Bad Gateway | Backend serverid pole kättesaadavad, kontrolli nende staatust |
| 503 Service Unavailable | Kõik backend serverid on maas |
| Timeout | Suurenda timeout väärtusi |
| SSL viga | Kontrolli sertifikaadi teed ja kehtivust |

---

## Levinud vead

- **Backend serverite IP vale** – kontrolli, et serverid on kättesaadavad
- **Tulemüür blokeerib** – backend serveritel peab port 80 olema avatud
- **Tervisekontroll ebaõnnestub** – backend peab tagastama HTTP 200
- **Sertifikaat puudu** – HAProxy vajab kombineeritud PEM faili
- **Moodulid lubamata** – Apache vajab `proxy`, `proxy_http`, `proxy_balancer` mooduleid

---

## Kasulikud lingid

**Teooria:**
- [NGINX Load Balancing Guide](https://tamerlan.dev/how-to-set-up-nginx-load-balancing-a-step-by-step-guide/)

**NGINX:**
- [DigitalOcean - NGINX Load Balancing](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing)
- [NGINX SSL Termination](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing-with-ssl-termination)

**HAProxy:**
- [HAProxy Load Balancer Setup](https://phoenixnap.com/kb/haproxy-load-balancer)
- [HAProxy SSL Termination](https://www.digitalocean.com/community/tutorials/how-to-implement-ssl-termination-with-haproxy-on-ubuntu-14-04)

**Apache:**
- [Apache Reverse Proxy](https://www.digitalocean.com/community/tutorials/how-to-use-apache-as-a-reverse-proxy-with-mod_proxy-on-ubuntu-16-04)
- [SSL Termination Guide](https://docs.digitalocean.com/products/networking/load-balancers/how-to/ssl-termination/)
