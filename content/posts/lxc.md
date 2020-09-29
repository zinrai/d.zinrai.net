---
title: "LXC"
date: 2020-09-29T17:50:53+09:00
tags: ['lxc', 'debian']
---

```
$ uname -a
Linux dbuster01 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64 GNU/Linux
$ cat /etc/debian_version
10.4
```

依存関係でいろいろ入ってしまうけれども iptables, dnsmasq, bridge-utils
それぞれを操作するのが面倒なので、 virsh (1) で全てよしなにやってもらうことにした。

```
% apt-get install libvirt-clients libvirt-daemon-system dnsmasq bridge-utils
```

```
% virsh net-list
 Name      State    Autostart   Persistent
 --------------------------------------------
  default   active   no          yes
```

```
# ip addr show virbr0
3: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 52:54:00:75:2f:a9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
```

LXC インストール

```
% apt-get install lxc
```

LXC コンテナの作成

```
% MIRROR=http://ftp.jp.debian.org/debian lxc-create -n sid64 -t debian -- -r sid
```

LXC のコンフィグについては lxc.container.conf (5) を参照

```
% cat /var/lib/lxc/sid64/config
# Template used to create this container: /usr/share/lxc/templates/lxc-debian
# Parameters passed to the template: -r sid
# Template script checksum (SHA-1): d5aa397522e36a17c64c014dd63c70d8607c9873
# For additional config options, please look at lxc.container.conf(5)

# Uncomment the following line to support nesting containers:
#lxc.include = /usr/share/lxc/config/nesting.conf
# (Be aware this has security implications)

#lxc.net.0.type = empty
lxc.apparmor.profile = generated
lxc.apparmor.allow_nesting = 1
lxc.rootfs.path = dir:/var/lib/lxc/sid64/rootfs

# Common configuration
lxc.include = /usr/share/lxc/config/debian.common.conf

# Container specific configuration
lxc.tty.max = 4
lxc.uts.name = sid64
lxc.arch = amd64
lxc.pty.max = 1024

lxc.net.0.type = veth
lxc.net.0.flags = up
lxc.net.0.name = eth0
lxc.net.0.link = virbr0
```
