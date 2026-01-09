# AlmaLinux Staatiline IP + Firewalld + NGINX

> See juhend näitab, kuidas seadistada AlmaLinuxil staatilist IP-d, firewalld tulemüüri ja NGINX veebiserverit.

[Tagasi README](README.md) · [← Eelmine](06-apache-https.md) · [Järgmine →](08-nginx.md)

---

## Eeldused

- AlmaLinux 9.x on paigaldatud
- Kasutajal on sudo õigused
- Baasmääratlused on tehtud

---

## Staatiline IP-aadress

```bash
sudo nmtui
```

Tegevused:
1. **Edit a connection** → vali liides
2. Muuda IPv4 seaded Manual režiimile
3. Lisa IP, Gateway, DNS
4. Salvesta ja välju
5. **Activate a connection** → deaktiveeri ja aktiveeri

Kontrollimine:
```bash
ip a
```

---

## Firewalld seadistamine

```bash
sudo dnf update -y
sudo dnf install firewalld -y
sudo systemctl enable --now firewalld
```

Reeglite lisamine:
```bash
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

Reeglite kontrollimine:
```bash
sudo firewall-cmd --list-all
```

---

## NGINX paigaldamine

```bash
sudo dnf install nginx -y
sudo systemctl enable --now nginx
```

### Testsaidi loomine

```bash
sudo mkdir -p /usr/share/nginx/test
```

```bash
echo '<html>
<head><title>Test</title></head>
<body>
<h1>NGINX töötab!</h1>
<p>Tere tulemast AlmaLinux serverisse.</p>
</body>
</html>' | sudo tee /usr/share/nginx/test/index.html
```

---

## Kontrollimine

```bash
systemctl status nginx
curl http://localhost
```

---

## Vigade leidmine ja parandamine

| Probleem | Lahendus |
|----------|----------|
| Port lubamata | `firewall-cmd --list-services` |
| NGINX ei käivitu | `journalctl -u nginx` |
| Sait ei ilmu | Kontrolli DocumentRoot teed |

---

## Levinud vead

- **nmtui muudatusi ei aktiveerita** – peale salvestamist tuleb ühendus uuesti aktiveerida
- **Firewalld reeglid lisamata** – HTTP/HTTPS peavad olema lubatud
- **SELinux blokeerib** – kontrolli `sudo ausearch -m avc`
