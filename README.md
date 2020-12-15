# fail2ban-glpi-mikrotik
Fail2ban for GLPI and Mikrotik russian user and locate

=== Install fail2ban ===

git clone https://github.com/fail2ban/fail2ban.git
cd fail2ban
python setup.py install

=== Add filter glpi ===
/etc/fail2ban/filter.d/glpi.conf
[INCLUDES]
before = common.conf
[Definition]
failregex =  Неудачная попытка входа пользователя \w+ с IP <HOST>
ignoreregex =

=== Add new action and filter in jail.conf ===
/etc/fail2ban/jail.conf
[glpi]
enabled = true
filter = glpi
port = http, https
logpath = /var/www/glpi/files/_log/event.log
maxretry = 3
action = mikrotik

=== Add new action Mikrotik ===
/etc/fail2ban/action.d/mikrotik.conf
[Definition]
actionstart =
actionstop =
actioncheck =
actionban = mikrotik ":ip firewall address-list add list=Fail2Ban address=<ip> comment=AutoFail2ban-<ip>"
actionunban = mikrotik ":ip firewall address-list remove [:ip firewall address-list find comment=AutoFail2ban-<ip>]”

=== Generate ssh-key for mikrotik ===

linux@ssh-keygen -t rsa
mikrotik@user add name=linux address=LINUX-SERVER-IP-ADDRESS group=full
scp /root/.ssh/id_rsa.pub admin@IP-ADDRESS-MIKROTIK
user ssh-keys import public-key-file=id_rsa.pub user=linux

=== Create bin file for usgate ===

touch /usr/bin/mikrotik

#!/bin/bash 						
ssh -l linux -p22 -i /root/.ssh/id_dsa MIKROTIK-IP-ADDRESS "$1»

Chmod +x /usr/bin/mikrotik

=== Add rules in filter in mikrotik ===
/ip firewall filter add action=reject chain=forward comment="fail2ban GLPI" dst-port=80,443 in-interface=ether-inet protocol=tcp reject-with=tcp-reset src-address-list=Fail2Ban




