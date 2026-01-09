# NTP (systemd-timesyncd)
[Tagasi README](README.md) · [← Eelmine](16-iscsi.md) · [Järgmine →](18-zabbix.md)

## Server
```bash
sudo apt install -y systemd-timesyncd
sudo timedatectl set-timezone Europe/Tallinn
sudo tee /etc/systemd/timesyncd.conf > /dev/null <<'EOF'
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org
FallbackNTP=2.pool.ntp.org
EOF
sudo systemctl enable systemd-timesyncd
sudo systemctl restart systemd-timesyncd
sudo ufw allow 123/udp
```
Kontroll:
```bash
timedatectl status
timedatectl show-timesync --all
```

## Klient (Debian)
```bash
sudo apt install -y systemd-timesyncd
sudo timedatectl set-timezone Europe/Tallinn
sudo tee /etc/systemd/timesyncd.conf > /dev/null <<'EOF'
[Time]
NTP=<NTP_SERVER_IP>
EOF
sudo systemctl enable systemd-timesyncd
sudo systemctl restart systemd-timesyncd
```

## Klient (Windows)
```powershell
tzutil /s "FLE Standard Time"
w32tm /config /manualpeerlist:"<NTP_SERVER_IP>" /syncfromflags:manual /update
net stop w32time; net start w32time
w32tm /query /status
w32tm /query /configuration
w32tm /resync
```

## Vigade leidmine ja parandamine
- Ajanihe: kontrolli timezone

## Levinud vead
- UDP/123 blokeeritud
