# HOWTO build Debian package from source

Sometimes it is needed to backport the fresh version of the package to the old
version of the Debian release. It can be done by building the package from the
newer Debian release source file. May be you can just use `apt source` to get
the package source but this note is about how to do it manually. Let's say you
want to build `dnsmasq` package from the Debian Trixie for the older release.

First I naively got the source from the Debian package directory unpacked it
and ran `debuild` script. But the only thing I got is the following error:
```
dh_auto_clean -p dnsmasq-utils -D contrib/lease-tools
Use of uninitialized value $abspath in -d at /usr/share/perl5/Debian/Debhelper/Buildsystem.pm line 140.
dh_auto_clean: error: invalid or non-existing path to the source directory: contrib/lease-tools
```
I tried moving source in different places to figure out the proper file
hierarchy but nothing works. Finally I used `apt source` to get the working
file hierarchy for old version of the package. The instruction below explains
how to build it without using `apt source` and finally build the new version of
the package.


## Prerequisites

First you need to install Debian package building system:
```
sudo apt-get install build-essential fakeroot devscripts
```

## Get the source code

Make directory to put source into:
```
mkdir -p src/debian
cd src/debian
```

Get the original source code and Debian build configuration from the [Debian
package directory](https://packages.debian.org/trixie/dnsmasq):
```
wget http://deb.debian.org/debian/pool/main/d/dnsmasq/dnsmasq_2.91-1.debian.tar.xz
wget http://deb.debian.org/debian/pool/main/d/dnsmasq/dnsmasq_2.91.orig.tar.gz
```

Unpack the original source code and Debian scripts to the one level under the
original source code:
```
tar xzf dnsmasq_2.91.orig.tar.gz
cd dnsmasq-2.91/
tar xJf ../dnsmasq_2.91-1.debian.tar.xz
```

Apply all Debian patches to the original source code:
```
for i in $(cat ./debian/patches/series); do echo $i; patch -p1 <./debian/patches/$i; done
```

## Build the package

Now use `debuild` tool to build the package.
```
debuild -b -uc -us
```

It may happen it will show you the list of the dependencies which are not
installed yet. Then you need to copy this list and install it using:
```
sudo apt-get install <list of dependencies>
```

## Links

[Debian Wiki BuildingTutorialSimplified](https://wiki.debian.org/BuildingTutorialSimplified)
