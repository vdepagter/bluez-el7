# bluez-5.49-3.el7.x86_64

This is a [RPM-release]([url](https://github.com/vdepagter/bluez-el7/releases)) for [bluez]([url](http://www.bluez.org))-5.49-3 built for CentOS 7.
It was required for getting Bluetooth integration working in combination with Home Assistant on CentOS 7. See [Github issue]([url](https://github.com/home-assistant/core/issues/76234)).
There may be some dependencies that need resolving, I will update instructions here as we discover them.
I took the source RPM from [Fedora Project]([url](https://koji.fedoraproject.org/koji/buildinfo?buildID=1074145)).

Github issue: https://github.com/home-assistant/core/issues/76234
BlueZ: http://www.bluez.org
RPM-release: https://github.com/vdepagter/bluez-el7/releases
Fedora Project build page for bluez-5.49-3.fc27: https://koji.fedoraproject.org/koji/buildinfo?buildID=1074145
Special thanks: https://budimir.cc/2022/01/08/compile-bluez-from-source/ and many others

Install with: ``rpm -ivh https://github.com/vdepagter/bluez-el7/releases/download/v5.49-3.el7/bluez-5.49-3.el7.x86_64.rpm``, add ``--nodeps`` if necessary.

It may also be worth giving bluez latest/stable snap a try. 20/stable didn't work for me.
```
[root@centos ~]# snap info bluez
name:      bluez
summary:   Bluetooth tools and daemons
publisher: Canonical✓
store-url: https://snapcraft.io/bluez
contact:   https://code.launchpad.net/~snappy-hwe-team/snappy-hwe-snaps/+git/bluez
license:   GPL-2.0
description: |
  This package contains tools and system daemons for using Bluetooth
  devices. BlueZ is the official Linux Bluetooth protocol stack. It
  is an Open Source project distributed under GNU General Public
  License (GPL). See the project homepage for more details:
  https://code.launchpad.net/~snappy-hwe-team/snappy-hwe-snaps/+git/bluez
snap-id: JmzJi9kQvHUWddZ32PDJpBRXUpGRxvNS
channels:
  20/stable:        5.53-7     2022-06-28 (334) 5MB -
  latest/stable:    **5.48-4**     2022-02-28 (314) 4MB -
  22/stable:        5.64-2     2022-05-03 (331) 4MB -
```

Below is a dump of most commands from my build attempt adventures, if you would like to try to rebuild it yourself.
You may also need sections for dependencies. Please let me know so I can update the instruction.
Executed from ``sudo su -``, you can also add ``sudo`` before the commands where needed.
```
#yum groupinstall -y "Development Tools"
yum install -y centos-release-scl
yum install -y devtoolset-11-gcc*
scl enable devtoolset-11 bash

yum install -y gcc glibc glibc-common gd gd-devel
yum install -y glib2-devel dbus-devel libudev-devel libical-devel libusb-devel #readline-devel
```

Build json-c dependency
```
cd /usr/local/src
wget https://s3.amazonaws.com/json-c_releases/releases/json-c-0.15.tar.gz
tar zxf json-c-0.15.tar.gz && cd json-c-0.15
mkdir build && cd build
# edit ../tests/CMakeLists.txt
# - comment out:
#   #target_sources(${TESTNAME} PRIVATE ../strerror_override.c)
# - change from: add_executable(${TESTNAME} ${TESTNAME}.c)
#   change to  : add_executable(${TESTNAME} ${TESTNAME}.c
#        "../strerror_override.c")
cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DBUILD_STATIC_LIBS=OFF ..
make && make install
```

Build ell dependency
```
cd /usr/local/src
wget https://mirrors.edge.kernel.org/pub/linux/libs/ell/ell-0.20.tar.xz
tar xf ell-0.20.tar.xz
cd ell-0.20/
./configure --prefix=/usr && make && make install
```

Building readline dependency
```
cd /usr/local/src
wget https://ftp.gnu.org/gnu/readline/readline-7.0.tar.gz
tar zxf readline-7.0.tar.gz
cd readline-7.0
./configure && make && make install
```

Building a recent release from source
Skip this, it didn't work, although you may want to try adjusting for building the same version source instead of from fc27 SRCRPM
```
cd /usr/local/src
wget https://mirrors.edge.kernel.org/pub/linux/bluetooth/bluez-5.63.tar.xz
tar xf bluez-5.63.tar.xz && cd bluez-5.63
./configure --enable-mesh --enable-testing --enable-tools --prefix=/usr --mandir=/usr/share/man --sysconfdir=/etc --localstatedir=/var
make && make install
```

Actually building from the SRC RPM (I have been advised one should build this as non-root ...)
```
yum install -y rpm-build cups-devel
rpm -ivh bluez-5.49-3.fc27.src.rpm
cd /root/rpmbuild/SPECS
# try without --nodeps first
rpmbuild -ba bluez.spec #--nodeps
```
