```bash
POSTFIX_REPLAYHOST="127.0.0.1"
POSTFIX_MYHOSTNAME="`hostname -f`"

MAIL_ENV="Production"
MAIL_HOST="host1"

postconf -e "myhostname = $POSTFIX_MYHOSTNAME"
postconf -e "mydomain = $myhostname"
postconf -e "myorigin = $mydomain"
postconf -e "inet_protocols = ipv4"
postconf -e "relayhost = $POSTFIX_REPLAYHOST"
postconf -e "smtp_generic_maps = hash:/etc/postfix/generic"
postconf -e "smtp_header_checks = regexp:/etc/postfix/header_checks"

cat >/etc/postfix/generic << EndOfMessage
root@`hostname -f`    root@`hostname -f`
zabbix-agent@`hostname -f`    zabbix-agent@`hostname -f`
zabbix@`hostname -f`    zabbix@`hostname -f`
EndOfMessage

chmod 644 /etc/postfix/generic

cat >/etc/postfix/header_checks  << EndOfMessage
/^From: root@`hostname -f`/ REPLACE From: "$MAIL_ENV ($MAIL_HOST)" <root@`hostname -f`>
/^From: zabbix-agent@`hostname -f`/ REPLACE From: "$MAIL_ENV ($MAIL_HOST) Zabbix Agent" <zabbix-agent@`hostname -f`>
/^From: zabbix@`hostname -f`/ REPLACE From: "$MAIL_ENV ($MAIL_HOST) Zabbix" <zabbix@`hostname -f`>
EndOfMessage

chmod 644 /etc/postfix/header_checks

rm -f /etc/postfix/generic.db

postmap /etc/postfix/generic
postmap -q "root@`hostname -f`" /etc/postfix/generic

service postfix restart && sleep 2 && echo "Message" | mailx -s "Subject" root@hostname.localdomain
```
