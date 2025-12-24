---
title: "FreeBSD 標準の jail.conf(5) を活用した jail 管理ツールを作ってみた"
date: 2025-12-22T16:04:41+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 3 22日目の記事です。

FreeBSD 標準の jail.conf(5) を活用した jail 管理ツールを作ってみました。

[zinrai/jailconf-builder](https://github.com/zinrai/jailconf-builder)

Go のテンプレートエンジンで jail.conf を記述し、 JSON でテンプレートに埋めるパラメータを記述して jail.conf(5) を /etc/jail.conf.d に生成します。 jail に対する操作は jail , jexec , jls などを使い jailconf-builder は jail.conf(5) の生成と base の展開が責務となっています。 Go のテンプレートで表現できる範囲であれば jail.conf(5) をどのようにでも表現できるようになっています。

## 使い方

[zinrai/jailconf-builder](https://github.com/zinrai/jailconf-builder) のバイナリをダウンロードし FreeBSD サーバーのパスの通った場所に配置します。

[Network Setup](https://github.com/zinrai/jailconf-builder?tab=readme-ov-file#network-setup) が完了している前提で使い方を説明していきます。

init サブコマンドで jailconf-builder を使うために必要なセットアップを行います。

```
% sudo jailconf-builder init
Confirmed directory exists: /etc/jail.conf.d
Created /etc/jail.conf with include directive.
Created directory: /var/jails
Created directory: /var/db/jailconf-builder/base
jailconf-builder initialized successfully.
% 
```

dl-base サブコマンドで jail に利用する base を取得します。

```
% sudo jailconf-builder dl-base -s https://download.freebsd.org/releases/amd64/14.3-RELEASE/base.txz
Downloading base.txz from https://download.freebsd.org/releases/amd64/14.3-RELEASE/base.txz...
Base system for FreeBSD 14.3-RELEASE downloaded successfully to /var/db/jailconf-builder/base/14.3-RELEASE/base.txz
%
```

create サブコマンドで jail.conf を生成して base を展開します。

```
% sudo jailconf-builder create -template examples/standard/jail.conf.tmpl -config examples/standard/jails.json
Extracting base system to /var/jails/myjail...
Jail 'myjail' created successfully.
  Config: /etc/jail.conf.d/1-myjail.conf
  Root:   /var/jails/myjail
%
```

delete サブコマンドで jail.conf と展開した base を削除します。

```
% sudo ./jailconf-builder delete -template examples/standard/jail.conf.tmpl -config examples/standard/jails.json -name myjail
Are you sure you want to delete jail 'myjail'? [y/N]: y
Deleted: /etc/jail.conf.d/1-myjail.conf
Deleted: /var/jails/myjail
Jail 'myjail' deleted successfully.
%
```

preview サブコマンドで生成される jail.conf を stdout できます。

```
freebsd@freebsd14-jail:~ % ./jailconf-builder preview -template examples/standard/jail.conf.tmpl -config examples/standard/jails.json -name myjail
# 1-myjail.conf
myjail {
    host.hostname = "myjail.jail";
    path = "/var/jails/myjail";

    vnet;
    vnet.interface = "epair1b";

    $ip4_addr = "192.168.2.11";
    $gw = "192.168.2.1";

    exec.prestart  = "ifconfig epair1 create up";
    exec.prestart += "ifconfig bridge0 addm epair1a";
    exec.start     = "ifconfig lo0 up 127.0.0.1";
    exec.start    += "ifconfig epair1b up $ip4_addr";
    exec.start    += "route add default $gw";
    exec.start    += "sh /etc/rc";
    exec.stop      = "sh /etc/rc.shutdown";
    exec.poststop  = "ifconfig epair1a destroy";

    mount.devfs;
    devfs_ruleset = 5;
    persist;
}
```

## なんで作ったのか

昔話です。 2012 年に無職だったときにサーバーと戯れるかパウンドケーキを作るかの日々を過ごしていました。そんなある日のこと [実践 FreeBSD サーバ構築・運用ガイド](https://www.amazon.co.jp/dp/4774150479) を眺めながら VIMAGE や jail と戯れる機会がありました。まだ Docker は世に出ていません。 FreeBSD 9.1-RELEAE から FreeBSD の標準として提供されはじめた [jail.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=jail.conf&apropos=0&sektion=5&manpath=FreeBSD+9.1-RELEASE&arch=default&format=html) に触れて「 jail.conf(5) に全ての jail の内容を記述するのではなく jail ごとに分割して jail.conf(5) を記述できるような仕組みがあったら jail.conf(5) を生成するツールを被せて FreeBSD が提供する標準の機能で jail が管理できる世界ができそうだな」と当時は思ってそこで終わりました。

2024 年に、いまの jail.conf(5) ってどんな風になっているのだろうとふと思い立って jail.conf(5) の man を眺めてみたところ /etc/jail.conf.d というディレクトリがあり include というパラメータで /etc/jail.conf.d に jail ごとの jail.conf(5) を管理できそうな雰囲気を感じ、 2012 年にやりたいと思っていたことがいまなら実現できそうだということで作ってみました。

ezjail , qjail ほか jail を管理するためのツールは沢山ありますが、どれもそのツールの使い方を覚えなければならず腰が重いという気持ちがあります。現時点で FreeBSD が標準で jail.conf(5) を10年以上提供しており jail.conf(5) に乗っかる薄いツールのほうが技術の入れ替わりが激しいなかで息の長いツールになるのではないかという期待も込めて作りました。
