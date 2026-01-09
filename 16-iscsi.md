# iSCSI
[Tagasi README](README.md) · [← Eelmine](15-squid.md) · [Järgmine →](17-ntp.md)

## Server
```bash
sudo apt update
sudo apt install -y targetcli-fb
sudo mkdir -p /srv/iscsi_disks
sudo dd if=/dev/zero of=/srv/iscsi_disks/disk01.img bs=1M count=1024
sudo targetcli /backstores/fileio create disk01 /srv/iscsi_disks/disk01.img
sudo targetcli /iscsi create iqn.2025-11.plaas.lan:target01
sudo targetcli /iscsi/iqn.2025-11.plaas.lan:target01/tpg1/luns create /backstores/fileio/disk01
sudo targetcli /iscsi/iqn.2025-11.plaas.lan:target01/tpg1/portals create 0.0.0.0 3260
sudo targetcli /iscsi/iqn.2025-11.plaas.lan:target01/tpg1/acls create iqn.1991-05.com.microsoft:desktop-ufajmra
sudo targetcli /iscsi/iqn.2025-11.plaas.lan:target01/tpg1/acls create iqn.2025-11.plaas.lan:debiancli
# autentimine (vastavalt vajadusele)
sudo targetcli /iscsi/iqn.2025-11.plaas.lan:target01/tpg1 set attribute authentication=0
sudo targetcli saveconfig && sudo targetcli exit
sudo systemctl enable rtslib-fb-targetctl && sudo systemctl start rtslib-fb-targetctl
sudo ufw allow 3260/tcp
```

## Klient (Debian)
```bash
sudo apt install -y open-iscsi
sudo sed -i 's/^InitiatorName=.*/InitiatorName=iqn.2025-11.plaas.lan:debianclient/' /etc/iscsi/initiatorname.iscsi
sudo systemctl enable --now iscsid
sudo iscsiadm -m discovery -t sendtargets -p <SERVER_IP>
sudo iscsiadm -m node -T iqn.2025-11.plaas.lan:target01 -p <SERVER_IP> --login
lsblk
# automaatne
automatic: sudo iscsiadm -m node -T iqn... --op update -n node.startup -v automatic
```

## Klient (Windows)
```powershell
iscsicli QAddTargetPortal <SERVER_IP> 3260
iscsicli ListTargets
iscsicli QLoginTarget iqn.2025-11.plaas.lan:target01
iscsicli SessionList
iscsicli PersistentLoginTarget iqn.2025-11.plaas.lan:target01 T 0 0 0
```
GUI: `iscsicpl.exe` → Discovery → Add Portal → Connect.

## Vigade leidmine ja parandamine
- Port 3260: kontrolli firewall
- `lsblk` ei näita: sessioon puudu

## Levinud vead
- Vale IQN
