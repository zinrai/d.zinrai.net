---
title: "cowbuilder"
date: 2020-09-30T00:27:00+09:00
tags: ['debian','cowbuilder']
---

仕事で様々なバージョンの Debian パッケージを作らなければならなかったとき、
環境を用意するコストがとても低くて重宝した。

```
% apt-get install cowbuilder
```

chroot 環境の作成

```
% cowbuilder --create --distribution buster --architecture amd64 --basepath  /var/cache/pbuilder/buster64.cow
```

chroot 環境へのログイン

```
# cowbuilder --login --basepath /var/cache/pbuilder/buster64.cow
```

--save オプションを付けると chroot 環境への変更を保存できる。

```
% cowbuilder --login --save --basepath /var/cache/pbuilder/buster64.cow
```

bind mount

```
% cowbuilder --login --save --bindmount /home/workdir --basepath /var/cache/pbuilder/buster64.cow
```

下記を実行し、作成した chroot 環境をまとめて更新していた。

```
% for a in /var/cache/pbuilder/*.cow; do cowbuilder --update --basepath $a; done
```
