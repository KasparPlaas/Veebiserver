# AlmaLinux staatiline IP + firewalld + NGINX
[Tagasi README](README.md) · [← Eelmine](06-apache-https.md) · [Järgmine →](08-nginx.md)

## IP seadistus
```bash
sudo nmtui
# Edit a connection → vali liides → salvesta
# Activate/Deactivate → uuesti aktiveeri
ip a
```

## Firewalld
```bash
sudo dnf update -y
sudo dnf install firewalld -y
sudo systemctl start firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

## NGINX
```bash
sudo dnf install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo mkdir -p /usr/share/nginx/test
printf '<html><body><h1>Test</h1><p>Töötab</p></body></html>' | sudo tee /usr/share/nginx/test/index.html
```

## Vigade leidmine ja parandamine
- Port lubamata: `firewall-cmd --list-services`

## Levinud vead
- `nmtui` muudatusi ei aktiveerita uuesti
