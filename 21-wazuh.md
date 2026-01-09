# Wazuh (Debian 12)
[Tagasi README](README.md) · [← Eelmine](20-nagios.md)

Minimalistlik lõpuprotsess: üksiku sõlme (manager + indexer + dashboard) paigaldus.

## Paigaldus (üks sõlm)
```bash
# Debian 12
curl -sO https://packages.wazuh.com/4.x/install.sh
sudo bash install.sh
# vali "Single-node"; sea paroolid (mäleta admin parooli)
```

## Teenused
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

## Dashboard
- Ava: https://<server_ip>
- Kasutaja: `admin`, parool: valisid installis

## Agent (Debian kliendil)
```bash
# Repo
wget -O - https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update && sudo apt install wazuh-agent -y
# Ühendus manageriga
sudo sed -i 's|<address>.*</address>|<address><SERVER_IP></address>|' /var/ossec/etc/ossec.conf
sudo systemctl enable --now wazuh-agent
```

## Firewall (vajadusel)
```bash
# Manager/Server
sudo ufw allow 1514/tcp   # agent
sudo ufw allow 1515/tcp   # agent registreerimine
sudo ufw allow 55000/tcp  # API
sudo ufw allow 443/tcp    # dashboard
```

## Vigade leidmine ja parandamine
- Dashboard ei avane: kontrolli `wazuh-dashboard` staatust + 443/tcp
- Agent ei liitu: kontrolli `1514/tcp`, `1515/tcp` ja `<address>`
- Indexer kollane: vaata `journalctl -u wazuh-indexer`

## Levinud vead
- Vale IP `<address>` väljal agendi `ossec.conf`
- Unustatud firewall reeglid
