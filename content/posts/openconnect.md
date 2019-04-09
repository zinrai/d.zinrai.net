---
title: "OpenConnect"
date: 2019-04-04T01:27:45+09:00
draft: false
---

GlobalProtect で VPN 接続する機会があり、 Mac も Windows も持っていないのでどうしたものかと調べていたところ、 OpenConnect というソフトウェアで GlobalProtect に VPN 接続することができることを知ったので記録として残しておく。 OpenConnect の開発者と Debian パッケージのメンテナーありがとう。

openconnect

https://packages.debian.org/ja/sid/openconnect

## 環境
```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux buster/sid
Release:        unstable
Codename:       sid
```

```
$ uname -a
Linux koyomi 4.19.0-4-amd64 #1 SMP Debian 4.19.28-2 (2019-03-15) x86_64 GNU/Linux
fchu@koyomi:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux buster/sid
Release:        unstable
Codename:       sid
```

```
$ apt-cache show openconnect
Package: openconnect
Version: 8.02-1
Installed-Size: 2573
Maintainer: Mike Miller <mtmiller@debian.org>
Architecture: amd64
Depends: vpnc-scripts, libc6 (>= 2.14), libgnutls30 (>= 3.6.5), libopenconnect5 (>= 8.01), libproxy1v5 (>= 0.4.14), libxml2 (>= 2.7.4)
Description-en: open client for Cisco AnyConnect, Pulse, GlobalProtect VPN
 OpenConnect is an SSL VPN client initially created to support Cisco's
 AnyConnect SSL VPN. It has since been extended to support the Pulse Connect
 Secure VPN (formerly known as Juniper Network Connect or Junos Pulse) and
 the Palo Alto Networks GlobalProtect SSL VPN.
 .
 A corresponding OpenConnect VPN server implementation can be found in the
 ocserv package.
Description-md5: c11aef69d31f0172dadbd4bc3375d349
Homepage: http://www.infradead.org/openconnect/
Tag: implemented-in::c, network::vpn, role::program
Section: net
Priority: optional
Filename: pool/main/o/openconnect/openconnect_8.02-1_amd64.deb
Size: 473132
MD5sum: d2433e28a8b4a9f2195937ac05d35f26
SHA256: 0a972888b9a2e3b863f2612f314584df862361ac796a5761f5b2e46492bf9622
```

## インストール
```
$ sudo apt-get install openconnect
```

## 接続
下記のコマンドライン実行後に ID, Password の入力を求められるので、入力すると VPN 接続が完了する。
```
$ sudo openconnect --protocol=gp vpn.example.jp
```
