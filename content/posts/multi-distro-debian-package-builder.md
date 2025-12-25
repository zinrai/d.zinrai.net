---
title: "複数ディストリビューション向けに Debian パッケージをビルドするツールを作ってみた"
date: 2025-12-24T05:00:15+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 24日目の記事です。

複数ディストリビューション向けに Debian パッケージをビルドするツールを作ってみました。

[zinrai/dpbuild](https://github.com/zinrai/dpbuild)


## 使い方

pbuilder や dpbuild をセットアップ済みの [Docker イメージ](https://github.com/zinrai/dpbuild?tab=readme-ov-file#docker) を提供しています。

pbuilder の環境をセットアップします。

```
$ mkdir ~/.dpbuild
$ cp config.yaml.example ~/.dpbuild/config.yaml
```

```
$ sudo docker run --rm --privileged \
  -v ~/.dpbuild:/root/.dpbuild \
  ghcr.io/zinrai/dpbuild init
Setting up environment for jammy (amd64)...
Executing: sudo pbuilder create --distribution jammy --architecture amd64 --mirror http://jp.archive.ubuntu.com/ubuntu --components main universe --basetgz /root/.dpbuild/pbuilder/jammy-amd64.tgz
W: cgroups are not available on the host, not using them.
I: Distribution is jammy.
I: Current time: Thu Dec 25 06:25:10 UTC 2025
I: pbuilder-time-stamp: 1766643910
I: Building the build environment
I: running debootstrap
/usr/sbin/debootstrap
I: Retrieving InRelease
...
```

ソースファイルを作成します。

```
$ cd example
$ sudo docker run --rm --privileged \
  -v $(pwd):/work \
  -w /work/hello \
  ghcr.io/zinrai/dpbuild source
Executing: tar cJf ../hello_1.0.0.orig.tar.xz --exclude ./debian --exclude ./.git --transform s,^\./,hello-1.0.0/, .
Executing: dpkg-source -b .
dpkg-source: info: using source format '3.0 (quilt)'
dpkg-source: info: building hello using existing ./hello_1.0.0.orig.tar.xz
dpkg-source: info: building hello in hello_1.0.0-1.debian.tar.xz
dpkg-source: info: building hello in hello_1.0.0-1.dsc
$
```

Debian パッケージを作成します。

```
$ sudo docker run --rm --privileged \
  -v ~/.dpbuild:/root/.dpbuild \
  -v $(pwd):/work \
  -w /work/hello \
  ghcr.io/zinrai/dpbuild package -dist bookworm -arch amd64 -dsc ../hello_1.0.0-1.dsc
Executing: sudo pbuilder build --basetgz /root/.dpbuild/pbuilder/bookworm-amd64.tgz --buildresult .packages/bookworm-amd64 ../hello_1.0.0-1.dsc
W: cgroups are not available on the host, not using them.
E: Neither ifconfig nor ip found.
N: Could not set up the loopback interface.
W: pbuilder: unshare CLONE_NEWNET not available
I: pbuilder: network access is available during build!
I: Current time: Thu Dec 25 06:44:52 UTC 2025
I: pbuilder-time-stamp: 1766645092
...
```

```
$ find hello
hello
hello/.packages
hello/.packages/bookworm-amd64
hello/.packages/bookworm-amd64/hello_1.0.0.orig.tar.xz
hello/.packages/bookworm-amd64/hello_1.0.0-1_source.changes
hello/.packages/bookworm-amd64/hello_1.0.0-1_amd64.deb
hello/.packages/bookworm-amd64/hello_1.0.0-1_amd64.changes
hello/.packages/bookworm-amd64/hello_1.0.0-1_amd64.buildinfo
hello/.packages/bookworm-amd64/hello_1.0.0-1.dsc
hello/.packages/bookworm-amd64/hello_1.0.0-1.debian.tar.xz
hello/go.mod
hello/main.go
hello/debian
hello/debian/source
hello/debian/source/format
hello/debian/control
hello/debian/copyright
hello/debian/changelog
hello/debian/rules
$
```

ビルドする環境を更新する場合は update サブコマンドを使います。

```
$ sudo docker run --rm --privileged \
  -v ~/.dpbuild:/root/.dpbuild \
  ghcr.io/zinrai/dpbuild update
Updating environment for jammy (amd64)...
Executing: sudo pbuilder update --basetgz /root/.dpbuild/pbuilder/jammy-amd64.tgz
W: cgroups are not available on the host, not using them.
I: Current time: Thu Dec 25 07:36:00 UTC 2025
I: pbuilder-time-stamp: 1766648160
I: Building the build Environment
I: extracting base tarball [/root/.dpbuild/pbuilder/jammy-amd64.tgz]
I: copying local configuration
W: No local /etc/mailname to copy, relying on /var/cache/pbuilder/build/17/etc/mailname to be correct
I: mounting /proc filesystem
...
```

## なんで作ったのか

内製アプリケーションを複数の Debian や Ubuntu 向けに Debian パッケージ化しなければならない状況がたまにあり、一対象ずつ pbuilder で Debian パッケージ化していくのが辛かったので pbuilder をバックエンドにしてコマンドを数回実行するだけで複数の Debian バージョンでビルドした Debian パッケージが作成できるようにしたくて作りました。
