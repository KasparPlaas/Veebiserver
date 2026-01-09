# SQUID Proxy Server

> Squid on võimas proxy ja vahemälu server, mis võimaldab kontrollida ja kiirendada veebiliiklust.

[Tagasi README](README.md) · [← Eelmine](14-nfs.md) · [Järgmine →](16-iscsi.md)

---

## Eeldused

- Debian/Ubuntu server on seadistatud
- Kasutajal on sudo õigused
- Port 3128 on vaba

---

## Paigaldamine

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install squid -y
sudo systemctl enable --now squid
```

Varukoopia:
```bash
sudo cp /etc/squid/squid.conf{,.bak}
```

---

## Põhikonfiguratsioon

```bash
sudo nano /etc/squid/squid.conf
```

Näidisseaded:
```text
# ACL definitsioonid
acl localnet src 10.0.80.0/24
acl SSL_ports port 443
acl Safe_ports port 80 443

# Ligipääsu reeglid
http_access allow localnet
http_access deny all

# Proxy port
http_port 3128

# Anonüümsus
forwarded_for off
request_header_access X-Forwarded-For deny all
request_header_access Via deny all
```

---

## Veebilehtede blokeerimine

### Domeenide blokeerimine

```bash
sudo nano /etc/squid/blocked-sites.txt
```

Sisu:
```text
.youtube.com
.twitter.com
.facebook.com
```

Lisa `squid.conf` faili:
```text
acl blocked_sites dstdomain "/etc/squid/blocked-sites.txt"
http_access deny blocked_sites
```

### Sõnade blokeerimine

```bash
sudo nano /etc/squid/blocked-words.txt
```

Sisu:
```text
gaming
gambling
torrent
```

Lisa `squid.conf` faili:
```text
acl blocked_words url_regex -i "/etc/squid/blocked-words.txt"
http_access deny blocked_words
```

Taaskäivitamine:
```bash
sudo systemctl restart squid
```

---

## Tulemüür

```bash
sudo ufw allow 3128/tcp
```

---

## Parooliga autentimine

```bash
sudo apt install apache2-utils -y
sudo htpasswd -c /etc/squid/passwd kasutaja
```

Lisa `squid.conf` faili:
```text
auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic realm Squid Proxy
acl authenticated proxy_auth REQUIRED
http_access allow authenticated
```

---

## SARG (Squid Analysis Report Generator)

```bash
sudo apt install sarg -y
sudo nano /etc/sarg/sarg.conf
```

Seaded:
```text
access_log /var/log/squid/access.log
output_dir /var/www/html/sarg
```

Raporti genereerimine:
```bash
sudo sarg
```

---

## Klientide seadistamine

### Brauseris
Seaded → Proxy → Manual → `server-ip:3128`

### Süsteemitasandil (Linux)

```bash
sudo nano /etc/environment
```

```text
http_proxy="http://server-ip:3128"
https_proxy="http://server-ip:3128"
```

### APT proxy

```bash
sudo nano /etc/apt/apt.conf.d/proxy.conf
```

```text
Acquire::http::Proxy "http://server-ip:3128";
Acquire::https::Proxy "http://server-ip:3128";
```

---

## Logide vaatamine

```bash
sudo tail -f /var/log/squid/access.log
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| ACL ei tööta | Kontrolli reeglite järjekorda |
| Ligipääs keelatud | `http_access allow` peab olema enne `deny all` |
| Klient ei ühendu | Kontrolli port 3128 tulemüüris |

---

## Levinud vead

- **http_access reeglite järjekord** – Squid töötleb reegleid järjekorras, esimene sobiv võidab
- **Vale ACL süntaks** – kontrolli tühikuid ja jutumärke
- **Proxy pole kliendis seadistatud** – iga rakendus vajab eraldi seadistust
