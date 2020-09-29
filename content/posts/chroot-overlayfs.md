---
title: "chroot + OverlayFS"
date: 2020-09-28T22:59:31+09:00
tags: ['debian', 'chroot', 'overlayfs']
---

出番がないかもしれないが、 OverlayFS で chroot 環境をクリーンに保つをやってみた。

```
$ uname -a
Linux dbuster01 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64 GNU/Linux
$ cat /etc/debian_version
10.4
```

```
% debootstrap buster ./buster64 http://ftp.jp.debian.org/debian
```

OverlayFS でのマウントについては mount (8) を参照

```
% mkdir upper workdir
% mount -t overlay overlay -o lowerdir=./buster64,upperdir=./upper,workdir=./workdir ./buster64
```

```
% df
ファイルシス   1K-ブロック    使用   使用可 使用% マウント位置
udev               2005984       0  2005984    0% /dev
tmpfs               404160    5436   398724    2% /run
/dev/vda3         99016264 2958444 91008396    4% /
tmpfs              2020792       0  2020792    0% /dev/shm
tmpfs                 5120       0     5120    0% /run/lock
tmpfs              2020792       0  2020792    0% /sys/fs/cgroup
overlay           99016264 2958444 91008396    4% /root/buster64
```

```
% mount --rbind /dev ./buster64/dev
% mount -t proc none ./buster64/proc
% mount --rbind /sys ./buster64/sys
% chroot ./buster64 /bin/bash
```
