---
title: "OverlayFS を活用した chroot 環境を管理するツールを作ってみた"
date: 2025-12-21T17:24:42+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 21日目の記事です。

OverlayFS を活用した chroot 環境を管理するツールを作ってみました。

[zinrai/chroot-prep](https://github.com/zinrai/chroot-prep)

## 使い方

chroot 用の環境作成

```
$ sudo debootstrap --arch=amd64 trixie trixie-amd64 http://deb.debian.org/debian
```

chroot 環境の利用と停止

```
$ sudo chroot-prep setup -dir trixie-amd64
Successfully set up chroot environment at /home/nanashi/trixie-amd64
$ sudo chroot trixie-amd64 /bin/bash
# cat /etc/debian_version
13.2
# exit
$ sudo chroot-prep cleanup -dir trixie-amd64
Successfully cleaned up chroot environment at /home/nanashi/trixie-amd64
$
```

OverlayFS を利用した chroot 環境の利用と停止

```
$ sudo chroot-prep setup -dir trixie-amd64 -overlay
Successfully set up overlay chroot environment
Base: /home/nanashi/trixie-amd64
Overlay: /home/nanashi/trixie-amd64.overlay
Use: sudo chroot /home/nanashi/trixie-amd64.overlay/merged
$ sudo chroot /home/nanashi/trixie-amd64.overlay/merged /bin/bash
# cat /etc/debian_version
13.2
# exit
exit
$ sudo chroot-prep cleanup -dir trixie-amd64 -overlay
Successfully cleaned up overlay 'overlay' at /home/nanashi/trixie-amd64
$ sudo find trixie-amd64.overlay/
trixie-amd64.overlay/
trixie-amd64.overlay/work
trixie-amd64.overlay/work/work
trixie-amd64.overlay/upper
trixie-amd64.overlay/upper/etc
trixie-amd64.overlay/upper/root
trixie-amd64.overlay/upper/root/.bash_history
trixie-amd64.overlay/merged
$
```

OverlayFS では -dir で指定した環境を read only にして -overlay で指定した名前のディレクトリを書き込むための領域とするため、書き込む領域を削除すれば元の状態に戻すことができます。 -overlay オプションは名前を指定できるようになっており、 chroot 環境のディスク消費を抑えながら用途ごとの環境を構築できるようにしています。

```
$ sudo chroot-prep setup -dir trixie-amd64 -overlay projectA
Successfully set up overlay chroot environment
Base: /home/nanashi/trixie-amd64
Overlay: /home/nanashi/trixie-amd64.projectA
Use: sudo chroot /home/nanashi/trixie-amd64.projectA/merged
$ sudo chroot trixie-amd64.projectA/merged /bin/bash
# cat /etc/debian_version
13.2
# exit
exit
$ sudo chroot-prep cleanup -dir trixie-amd64 -overlay projectA
Successfully cleaned up overlay 'projectA' at /home/nanashi/trixie-amd64
$ sudo find trixie-amd64.projectA/
trixie-amd64.projectA/
trixie-amd64.projectA/work
trixie-amd64.projectA/work/work
trixie-amd64.projectA/upper
trixie-amd64.projectA/upper/etc
trixie-amd64.projectA/upper/root
trixie-amd64.projectA/upper/root/.bash_history
trixie-amd64.projectA/merged
$
```

## なんで作ったのか

コンテナの時代になんで chroot の話なのかとは思うかもしれませんが、私はいまでも chroot 環境を使うことがあります。 chroot に /proc , /dev , /sys を mount したり umount したりという操作が面倒で chroot 用のディレクトリを指定したら chroot dir /bin/bash できる状態にしたくて作りました。

2012年頃に Union mount の実装である [aufs](https://aufs.sourceforge.net) を使って read only の領域と書き込む領域を重ね合わせて chroot 環境を構成するということをやって遊んでいました。現在は Linux Kernel で提供されている [OverlayFS](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html) がスタンダードだろうと考え aufs を使い実装していたことを OverlayFS を使って実装してみました。
