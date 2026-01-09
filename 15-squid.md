# SQUID proxy
[Tagasi README](README.md) · [← Eelmine](14-nfs.md) · [Järgmine →](16-iscsi.md)

## Install
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install squid -y
sudo systemctl enable --now squid
sudo cp /etc/squid/squid.conf{,.bak}
```

## Põhiseadistus
```bash
sudo nano /etc/squid/squid.conf
# acl my_list src 10.0.80.0/24
# http_access allow my_list
# http_access deny plaas.lan
# acl plaas.lan dstdomain "/etc/squid/soovimatud-saidid"
# request_header_access Referer deny all
# request_header_access X-Forwarded-For deny all
# request_header_access Via deny all
# request_header_access Cache-Control deny all
# forwarded_for off
```

Keelatud saidid:
```bash
sudo nano /etc/squid/soovimatud-saidid
# .youtube.com
# .twitter.com
# .redhat.com
```

Keelatud sõnad:
```bash
sudo nano /etc/squid/banned-words
# gaming
# porno
# film
# mp3
# mp4
# ja lisa squid.conf:
# acl banned-words url_regex "/etc/squid/banned-words"
```

Restart + firewall:
```bash
sudo systemctl restart squid
sudo ufw allow 3128/tcp
```

Logid:
```bash
sudo tail -f /var/log/squid/access.log
```

## Parooliga kasutamine
```bash
sudo apt install apache2-utils -y
sudo htpasswd -c /etc/squid/passwd <username>
# squid.conf lisa:
# auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/passwd
# auth_param basic children 5
# auth_param basic realm Squid Basic Authentication
# auth_param basic credentialsttl 2 hours
# acl auth_users proxy_auth REQUIRED
# http_access allow auth_users
sudo systemctl restart squid
```

## SARG
```bash
sudo apt install sarg
sudo nano /etc/sarg/sarg.conf
# access_log /var/log/squid/access.log
# ip_resolv on
sudo sarg
```

## Klientide seadistused
Firefox/Chrome: GUI kaudu käsitsi.
APT/curl/wget/süsteem:
```bash
# /etc/apt/apt.conf
Acquire::http::Proxy "http://<proxy_ip>:3128";
Acquire::https::Proxy "https://<proxy_ip>:3128";
# ~/.curlrc
proxy = http://<proxy_ip>:3128
https_proxy=http://<proxy_ip>:3128
# ~/.wgetrc
http_proxy=http://<proxy_ip>:3128
https_proxy=http://<proxy_ip>:3128
# /etc/profile.d/squid.sh
export proxy_ip=192.168.200.56
export http_proxy=http://$proxy_ip:3128
export https_proxy=http://$proxy_ip:3128
export ftp_proxy=http://$proxy_ip:3128
export HTTP_PROXY=http://$proxy_ip:3128
export HTTPS_PROXY=http://$proxy_ip:3128
export FTP_PROXY=http://$proxy_ip:3128
source /etc/profile.d/squid.sh
```

## Vigade leidmine ja parandamine
- `ngrep -d any port 3128` aitab kontrollida liiklust

## Levinud vead
- `http_access` reeglite järjekord vale
