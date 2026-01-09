# Nagios
[Tagasi README](README.md) · [← Eelmine](19-cacti.md) · [Järgmine →](21-wazuh.md)

## Install + CGI
```bash
sudo apt -y install nagios4
sudo nano /etc/nagios4/cgi.cfg
# use_authentication=1
sudo nano /etc/apache2/conf-enabled/nagios4-cgi.conf
# Require ip 10.0.80.0/24
# AuthDigestDomain "Nagios4"
# AuthUserFile "/etc/nagios4/htdigest.users"
# Require valid-user
sudo a2enmod cgid
sudo systemctl restart nagios4 apache2
htdigest /etc/nagios4/htdigest.users "Nagios4" nagiosadmin
```
UI: `http://<ip>/nagios4`.

## Hostide näited
AlmaLinux:
```bash
sudo nano /etc/nagios4/servers/alma.cfg
# define host { use linux-server; host_name alma01; alias AlmaLinux Client; address 10.0.80.3 }
# define service { use generic-service; host_name alma01; service_description PING; check_command check_ping!100.0,20%!500.0,60% }
# define service { use generic-service; host_name alma01; service_description Disk Usage; check_command check_nrpe!check_disk }
# define service { use generic-service; host_name alma01; service_description Load; check_command check_nrpe!check_load }
```
Windows:
```bash
sudo nano /etc/nagios4/servers/winklient.cfg
# define host { use windows-server; host_name win01; alias Windows Client; address 192.168.1.30 }
# define service { use generic-service; host_name win01; service_description PING; check_command check_ping!100.0,20%!500.0,60% }
# define service { use generic-service; host_name win01; service_description CPU Load; check_command check_nt!CPULOAD!-l 5,80,90 }
# define service { use generic-service; host_name win01; service_description Disk C; check_command check_nt!USEDDISKSPACE!-l c -w 80 -c 90 }
```

Test + restart:
```bash
sudo nagios4 -v /etc/nagios4/nagios.cfg
sudo systemctl restart nagios4
```

## Vigade leidmine ja parandamine
- `nagios4 -v` annab täpse vea

## Levinud vead
- Vale `check_command` parameetrid
