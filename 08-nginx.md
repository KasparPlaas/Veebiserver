# NGINX hosti näidis
[Tagasi README](README.md) · [← Eelmine](07-almalinux-staatiline-ip.md) · [Järgmine →](09-smtp-postfix.md)

## Hosti fail
```bash
cd /etc/nginx/conf.d/
sudo nano test-host.conf
```
Sisu:
```nginx
server {
    listen       80;
    listen       [::]:80;
    server_name  example.com;
    root         /usr/share/nginx/example;
    index        index.html index.htm;
    access_log   /var/log/nginx/example.com_access.log;
    error_log    /var/log/nginx/example.com_error.log;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

## Vigade leidmine ja parandamine
- Test: `nginx -t` ja `systemctl reload nginx`

## Levinud vead
- Vale `root` kaust
