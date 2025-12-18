---
title: "iptables の設定を YAML で管理するツールを作ってみた"
date: 2025-12-18T13:25:23+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 18日目の記事です。

iptables の設定を YAML で管理するツールを作ってみました。

[zinrai/yptables](https://github.com/zinrai/yptables)

## 使い方

```
$ cat examples/basic-features.yaml
tables:
  filter:
    chains:
      INPUT:
        policy: DROP
        rules:
          - protocol: tcp
            match:
              - name: tcp
                options:
                  dport: "22"
            jump: ACCEPT

          - protocol: tcp
            match:
              - name: multiport
                options:
                  dports: "80,443"
              - name: comment
                options:
                  comment: "Web traffic"
            jump: WEB

      WEB:
        rules:
          - source: 10.0.0.0/8
            match:
              - name: comment
                options:
                  comment: "Allow internal network"
            jump: ACCEPT

          - match:
              - name: comment
                options:
                  comment: "Drop other web traffic"
            jump: DROP

  nat:
    chains:
      PREROUTING:
        rules:
          - protocol: tcp
            match:
              - name: tcp
                options:
                  dport: "80"
              - name: comment
                options:
                  comment: "Redirect HTTP to local proxy"
            jump: REDIRECT
            options:
              to-port: "8080"
```

```
$ yptables examples/basic-features.yaml
*filter
:INPUT DROP [0:0]
:WEB - [0:0]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m multiport --dports 80,443 -m comment --comment "Web traffic" -j WEB
-A WEB -s 10.0.0.0/8 -m comment --comment "Allow internal network" -j ACCEPT
-A WEB -m comment --comment "Drop other web traffic" -j DROP
COMMIT

*nat
-A PREROUTING -p tcp -m tcp --dport 80 -m comment --comment "Redirect HTTP to local proxy" -j REDIRECT
COMMIT

$
```

## 仕組み

yptables は iptables-restore コマンドに読み込ませることができる形式で iptables の設定を stdout します。

yptables から生成された iptables の設定を iptabels-restore コマンドでテストしたり、 iptables-restore コマンド経由で iptables の設定を読み込ませるオペレーションを想定しています。 yptables は iptables-restore コマンドが読める形式で iptables の設定を生成する責務までで、 iptables の設定のテストや適用は iptables が提供しているコマンドに任せる構成となっています。

```
$ yptables config.yaml > iptables.rules
$ sudo iptables-restore --test iptables.rules
$ sudo iptables-restore iptables.rules
```

YAML で表現できるのは filter と nat テーブルで mangle テーブルには対応していません。

## なんで作ったのか

YAML と HCL ( Hashicorp Configuration Language ) での構成管理に触れる機会が多いと感じていて、 iptables を YAML で構成管理する yptables というツール名が頭に浮かび、ワイピーテブルスという語呂がよいなと思ったことから作ってみました。
