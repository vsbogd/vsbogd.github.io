Administration:
- [HOWTO create new VirtualBox VM using command line](./administration/howto-create-vbox-vm.md)
- [HOWTO setup LDAP client](./administration/howto-setup-ldap-client.md)
- [HOWTO send test mail via SMTP](./administration/howto-send-mail-via-smtp.md)
- [HOWTO setup LXC container](./administration/howto-create-lxc-container.md)
- [HOWTO setup Exim4 to work with Gmail account](./administration/howto-send-mail-via-gmail.md)
- [HOWTO create app password to use Google
  services](https://support.google.com/accounts/answer/185833)
- [HOWTO test network bandwidth and latency](./administration/howto-test-network-bandwidth-and-latency.md)
- [HOWTO simulate network bandwidth and latency](./administration/howto-simulate-network-bandwidth-and-latency.md)
- [Generate OpenVPN configuration](https://github.com/vsbogd/openvpn-config-generator)
- Change OpenVPN key password inside config file:
  [openvpn-change-key-pass.sh](https://github.com/vsbogd/openvpn-config-generator/raw/refs/heads/main/openvpn-change-key-pass.sh)
- [HOWTO setup NAT and traffic forwarding via OpenVPN server](./administration/howto-forward-traffic-via-openvpn.md)
- [HOWTO reset Windows 10 password using Linux](./administration/reset-windows-password.md)
- [HOWTO make a Windows 10 and 11 bootable USB stick on Linux](./administration/make-windows-boot-usb-linux.md)

- Make wildcard include hidden files:
```sh
shopt -s dotglob
```
- Make .deb package from source:
```sh
# instead of `make install`
checkinstall
```
- Show SSL certificate details:
```sh
openssl x509 -in example.pem -text
```
- Get SSL certificates from server:
```sh
openssl s_client -showcerts -servername www.example.com -connect  www.example.com:443 </dev/null
```
- Use socat to interact with USB/UART adapter (Ctrl+X to exit socat):
```sh
socat -,raw,echo=0,escape=0x18 /dev/ttyUSB0,b115200,raw,echo=0
```
- Check keyboard and mouse X Window events:
```sh
xev
```
- Run a shell with limited amount of memory:
```sh
systemd-run -p MemoryMax=1G -p MemorySwapMax=0 --shell
```
- Add PulseAudio sink which combines all other sinks:
```sh
pactl load-module module-combine-sink
```

Coding:
- [Using GRPC in Java Maven project](./coding/using-grpc-in-java-maven-project.md)
- [HOWTO move files between Git repositories preserving history](./coding/move-files-between-git-repos-preserving-history.md)
- [HOWTO install Android SDK using command line tools](./coding/install-android-using-command-line.md)
- [HOWTO install rospy ROS Noetic module to Ubuntu 18.04](./coding/install-rospy-noetic-ubuntu-1804.md)
- [HOWTO use fluentd to post errors into Slack channel](./coding/use-fluentd-to-post-errors-to-slack.md)
- [HOWTO get specific revision from Git repo without getting history](./coding/get-specific-revision-from-git-repo.md)
- Convert under_score to camelCase by sed:
```sh
sed -E 's/([^ ])_(.)/\1\U\2/g'
```
- Grep binary string:
```sh
grep -RobUaP 'S\x00O\x00M\x00E\x00T\x00H\x00I\x00N\x00G' <dir>
```

Misc:
- [HOWTO capture VHS video using Pinnacle Dazzle DVC100 on Linux](./misc/howto-capture-vhs.md)
