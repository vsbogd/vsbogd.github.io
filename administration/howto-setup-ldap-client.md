# LDAP details

You need the following LDAP details:

* LDAP server: usually ldap://<ip>, for instance 192.168.1.1
* root domain: usually dc=<organization>,dc=<country>, for instance dc=exampleorg,dc=com

# Setup LDAP client

## PAM and NSS setup

Install necessary packages:

```
$ sudo apt-get install libnss-ldapd libpam-ldapd
```

After installation you should use LDAP server and root domain to configure client correctly.

## AutoFS setup

Install necessary packages:

```
$ sudo apt-get install autofs-ldap
```

Add the following text to the end of the /etc/default/autofs:

    MASTER_MAP_NAME="ldap://192.168.1.1/automountMapName=auto.master,ou=Mounts,dc=exampleorg,dc=com"
    LOGGING="verbose"
    SEARCH_BASE="ou=Mounts,dc=exampleorg,dc=com"
    
    MAP_OBJECT_CLASS="automountMap"
    ENTRY_OBJECT_CLASS="automount"
    MAP_ATTRIBUTE="automountMapName"
    ENTRY_ATTRIBUTE="automountKey"
    VALUE_ATTRIBUTE="automountInformation"

Restart autofs:

```
$ sudo systemctl restart autofs
```

## Homedir setup

At the moment automatic homedir creation doesn't work.
So you should manually create homedir for any user which should be able to login using Gnome.
Without homedir user can login using console only.

```
$ sudo cp -R /etc/skel /home/<username>
$ sudo chown -R <username>:<usergroup> /home/<username>
```
