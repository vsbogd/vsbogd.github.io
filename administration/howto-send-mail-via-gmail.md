# HOWTO setup exim4 to work with Gmail account

Setup Exim4 to use Gmail as an external SMTP server:
```
# dpkg-reconfigure exim4-config
```

Choose the  answers:
- Choose "mail sent by smarthost; received via SMTP or fetchmail"
- Set to "localhost" for "System mail name:".
- Set to "127.0.0.1" for "IP-addresses to listen on for incoming SMTP connections" to refuse external connections.
- Leave as empty for "Other destinations for which mail is accepted:".
- Leave as empty for "Machines to relay mail for:".
- Set to "smtp.gmail.com::587" for "IP address or host name of the outgoing smarthost:".
- Choose "NO" for "Hide local mail name in outgoing mail?".
- Choose "NO" for "Keep number of DNS-queries minimal (Dial-on-Demand)?".
- Choose "mbox format in /var/mail/" for "Delivery method for local mail".
- Choose "YES" for "Split configuration into small files?". 

Enable 2-factor authentication on your Gmail account used for sending emails
and follow Google instruction [to create app specific
password](https://support.google.com/mail/answer/185833?hl=en).

Edit `/etc/exim4/passwd.client` and add the following line inside:
```
smtp.gmail.com:<gmail-sender-account-name>:<your-app-password>
```
You should use account name not email address here.

If it is new or restored file then ensure the permissions are correct:
```
# chown root:Debian-exim /etc/exim4/passwd.client
# chmod 640 /etc/exim4/passwd.client
```

You can also configure replacement of the local sender address to your Gmail
one:
```
# echo '<local-user-name>: <gmail-sender-account-name>@gmail.com' >> /etc/email-addresses
# echo '<local-user-name>@localhost: <gmail-sender-account-name>@gmail.com' >> /etc/email-addresses
# echo '<local-user-name>@<hostname>: <gmail-sender-account-name>@gmail.com' >> /etc/email-addresses
```

Update Exim4 configuration, remove stale mail:
```
# update-exim4.conf
# invoke-rc.d exim4 restart
# exim4 -qff
```

Check that email is sent using:
```
$ echo "Subj" | mail -s "Test email from Exim4" <email-address>
```

Check logs to see if email was sent successfully:
```
# less /var/log/exim4/mainlog
```

Check if email is received by `<email-address>`. If it is Gmail address you may
need add filtering rule to prevent email be classified as a spam.

Links:
- [Debian Wiki Exim4 for Gmail](https://wiki.debian.org/Exim4Gmail)
- [Gmail mail client
  setup](https://support.google.com/mail/answer/7104828?hl=en&visit_id=638300159533347627-1330126653&rd=3)
- [Debian Wiki Exim4](https://wiki.debian.org/Exim)
