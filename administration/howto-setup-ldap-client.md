# HOWTO setup LDAP client

## LDAP details

You need the following LDAP details:

* LDAP server: usually `ldap://<ip>`, for instance `ldap://192.168.1.1`
* root domain: usually `dc=<organization>,dc=<country>`, for instance `dc=example.org,dc=com`

## PAM and NSS setup

Install necessary packages:

```
$ sudo apt-get install libnss-ldapd libpam-ldapd
```

Use LDAP server and root domain when configuring the client package during
installation.

To automatically create a home directory on login add the following string to
the end of the `/etc/pam.d/common-session` file (before the final comment):
```
session optional pam_mkhomedir.so skel=/etc/skel umask=0002
```

Restart NSS services:
```
$ sudo systemctl restart nscd
$ sudo systemctl restart nslcd
```

## AutoFS setup

Install necessary packages:

```
$ sudo apt-get install autofs-ldap
```

Add the following text to the end of the /etc/default/autofs:
```
    MASTER_MAP_NAME="ldap://192.168.1.1/automountMapName=auto.master,ou=Mounts,dc=exampleorg,dc=com"
    LOGGING="verbose"
    SEARCH_BASE="ou=Mounts,dc=exampleorg,dc=com"
    
    MAP_OBJECT_CLASS="automountMap"
    ENTRY_OBJECT_CLASS="automount"
    MAP_ATTRIBUTE="automountMapName"
    ENTRY_ATTRIBUTE="automountKey"
    VALUE_ATTRIBUTE="automountInformation"
```

Restart autofs:
```
$ sudo systemctl restart autofs
```

## References

PAM, NSS, AutoFS sorted by usefullness:
- [Debian handbook](https://debian-handbook.info/browse/stable/sect.ldap-directory.html)
- [Debian Wiki LDAP NSS](https://wiki.debian.org/LDAP/NSS)
- [Debian Wiki LDAP PAM](https://wiki.debian.org/LDAP/PAM)
- [HOWTO change password on LDAP server](https://www.digitalocean.com/community/tutorials/how-to-change-account-passwords-on-an-openldap-server)
- [Debian Wiki LDAP](https://wiki.debian.org/LDAP)
- [LDAP HOWTO](http://www.tldp.org/HOWTO/LDAP-HOWTO/)
- [HOWTO configure AutoFS in LDAP](http://sadiquepp.blogspot.com/2009/02/how-to-configure-autofs-maps-in-ldap.html)
- [AutoFS in LDAP](https://help.ubuntu.com/community/AutofsLDAP)

Kerberos related:
- [Debian Wiki LDAP Kerberos](https://wiki.debian.org/LDAP/Kerberos)
- [SpinLockSolutions Kerberos](http://techpubs.spinlocksolutions.com/dklar/kerberos.html)
- [SpinLockSolutions LDAP](http://techpubs.spinlocksolutions.com/dklar/ldap.html)
- [Ubuntu Kerberos](https://help.ubuntu.com/community/Kerberos)
- [Ubuntu LDAP client authentication](https://help.ubuntu.com/community/LDAPClientAuthentication)
