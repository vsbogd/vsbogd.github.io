Administration:
- [HOWTO create new VirtualBox VM using command line](./administration/howto-create-vbox-vm.md)
- [HOWTO setup LDAP client](./administration/howto-setup-ldap-client.md)
- [HOWTO send test mail via SMTP](./administration/howto-send-mail-via-smtp.md)
- [HOWTO setup LXC container](./administration/howto-create-lxc-container.md)
- [HOWTO setup exim4 to work with Gmail
  account](https://wiki.debian.org/Exim4Gmail)

Coding:
- [Using GRPC in Java Maven project](./coding/using-grpc-in-java-maven-project.md)
- Convert under_score to camelCase by sed:
```sh
sed -E 's/([^ ])_(.)/\1\U\2/g'
```

Network:
- Get SSL certificates from server:
```sh
openssl s_client -showcerts -servername www.example.com -connect  www.example.com:443 </dev/null
```
