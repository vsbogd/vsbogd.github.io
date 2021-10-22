Administration:
- [HOWTO create new VirtualBox VM using command line](./administration/howto-create-vbox-vm.md)
- [HOWTO setup LDAP client](./administration/howto-setup-ldap-client.md)
- [HOWTO send test mail via SMTP](./administration/howto-send-mail-via-smtp.md)
- [HOWTO setup LXC container](./administration/howto-create-lxc-container.md)
- [HOWTO setup exim4 to work with Gmail
  account](https://wiki.debian.org/Exim4Gmail)
- [HOWTO test network bandwidth and latency](./administration/howto-test-network-bandwidth-and-latency.md)
- Make wildcard include hidden files:
```sh
shopt -s dotglob
```
- Make .deb package from source:
```sh
# instead of `make install`
checkinstall
```

Coding:
- [Using GRPC in Java Maven project](./coding/using-grpc-in-java-maven-project.md)
- [HOWTO move files between Git repositories preserving history](./coding/move-files-between-git-repos-preserving-history.md)
- [HOWTO install Android SDK using command line tools](./coding/install-android-using-command-line.md)
- [HOWTO install rospy ROS Noetic module to Ubuntu 18.04](./coding/install-rospy-noetic-ubuntu-1804.md)
- [HOWTO use fluentd to post errors into Slack channel](./coding/use-fluentd-to-post-errors-to-slack.md)
- Convert under_score to camelCase by sed:
```sh
sed -E 's/([^ ])_(.)/\1\U\2/g'
```
- Grep binary string:
```sh
grep -RobUaP 'S\x00O\x00M\x00E\x00T\x00H\x00I\x00N\x00G' <dir>
```

Network:
- Get SSL certificates from server:
```sh
openssl s_client -showcerts -servername www.example.com -connect  www.example.com:443 </dev/null
```
