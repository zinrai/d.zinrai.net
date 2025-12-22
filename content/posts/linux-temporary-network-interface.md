---
title: "Linux にて検証用の一時的なネットワークインタフェースを設定するツールを作ってみた"
date: 2025-12-19T13:42:05+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 19日目の記事です。

Linux にて検証用の一時的なネットワークインタフェースを設定するツールを作ってみました。

[zinrai/netshed](https://github.com/zinrai/netshed)

## セットアップ

* [zinrai/netshed](https://github.com/zinrai/netshed) のバイナリをダウンロードしてパスの通った場所に配置します
* nft コマンドをインストールします 
    * Debian GNU/Linux の場合は apt install nftables でインストールできます

## 使い方

dummy interface の作成と削除

```
$ cat examples/dummy.yaml
networks:
  - name: "dummy0"
    type: "dummy"
    address: "10.0.0.1/24"
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 85046sec preferred_lft 85046sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86181sec preferred_lft 14181sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$ sudo netshed create -config examples/dummy.yaml
2025/12/22 10:58:40 created network dummy0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 85028sec preferred_lft 85028sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86163sec preferred_lft 14163sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
8: dummy0: <BROADCAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
    link/ether 9e:b3:b0:44:c1:7a brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.1/24 brd 10.0.0.255 scope global dummy0
       valid_lft forever preferred_lft forever
    inet6 fe80::8c8c:dff:fe5b:47ca/64 scope link
       valid_lft forever preferred_lft forever
$ sudo netshed delete -config examples/dummy.yaml
2025/12/22 10:58:49 deleted network dummy0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 85020sec preferred_lft 85020sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86155sec preferred_lft 14155sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$
```

bridge interface の作成と削除

```
$ cat examples/bridge-internal.yaml
networks:
  - name: "internal0"
    type: "bridge"
    subnet: "192.168.200.0/24"
    gateway: "192.168.200.1/24"
    masquerade: false
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84804sec preferred_lft 84804sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86383sec preferred_lft 14383sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$ sudo netshed create -config examples/bridge-internal.yaml
2025/12/22 11:02:29 created network internal0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84799sec preferred_lft 84799sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86379sec preferred_lft 14379sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
9: internal0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether fe:67:f5:09:dd:c7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.1/24 brd 192.168.200.255 scope global internal0
       valid_lft forever preferred_lft forever
    inet6 fe80::c869:d2ff:fe23:788a/64 scope link tentative
       valid_lft forever preferred_lft forever
$ sudo netshed delete -config examples/bridge-internal.yaml
2025/12/22 11:02:43 deleted network internal0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84785sec preferred_lft 84785sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86364sec preferred_lft 14364sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$
```

bridge interface と IP Masquerade の作成と削除

```
$ cat examples/bridge-masquerade.yaml
networks:
  - name: "vm0"
    type: "bridge"
    subnet: "192.168.100.0/24"
    gateway: "192.168.100.1/24"
    masquerade: true
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84619sec preferred_lft 84619sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86199sec preferred_lft 14199sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$ sudo nft list ruleset
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain LIMADNS {
                ip daddr 192.168.5.3 udp dport 53 counter packets 5 bytes 342 dnat to 192.168.5.2:39394
                ip daddr 192.168.5.3 tcp dport 53 counter packets 0 bytes 0 dnat to 192.168.5.2:34697
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                counter packets 8 bytes 571 jump LIMADNS
        }

        chain OUTPUT {
                type nat hook output priority -100; policy accept;
                counter packets 10 bytes 678 jump LIMADNS
        }

        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
        }
}
$ sudo netshed create -config examples/bridge-masquerade.yaml
2025/12/22 11:05:48 created network vm0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84601sec preferred_lft 84601sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86180sec preferred_lft 14180sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
10: vm0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 56:85:47:60:2b:2d brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global vm0
       valid_lft forever preferred_lft forever
    inet6 fe80::a0b9:f4ff:febc:ab0a/64 scope link tentative
       valid_lft forever preferred_lft forever
$ sudo nft list ruleset
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain LIMADNS {
                ip daddr 192.168.5.3 udp dport 53 counter packets 5 bytes 342 dnat to 192.168.5.2:39394
                ip daddr 192.168.5.3 tcp dport 53 counter packets 0 bytes 0 dnat to 192.168.5.2:34697
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                counter packets 8 bytes 571 jump LIMADNS
        }

        chain OUTPUT {
                type nat hook output priority -100; policy accept;
                counter packets 11 bytes 726 jump LIMADNS
        }

        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
                ip saddr 192.168.100.0/24 masquerade comment "netshed:vm0"
        }
}
$ sudo netshed delete -config examples/bridge-masquerade.yaml
2025/12/22 11:06:11 deleted network vm0
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:55:55:d6:0c:6f brd ff:ff:ff:ff:ff:ff
    inet 192.168.5.15/24 metric 200 brd 192.168.5.255 scope global dynamic eth0
       valid_lft 84577sec preferred_lft 84577sec
    inet6 fec0::5055:55ff:fed6:c6f/64 scope site dynamic mngtmpaddr noprefixroute
       valid_lft 86157sec preferred_lft 14157sec
    inet6 fe80::5055:55ff:fed6:c6f/64 scope link
       valid_lft forever preferred_lft forever
$ sudo nft list ruleset
# Warning: table ip nat is managed by iptables-nft, do not touch!
table ip nat {
        chain LIMADNS {
                ip daddr 192.168.5.3 udp dport 53 counter packets 5 bytes 342 dnat to 192.168.5.2:39394
                ip daddr 192.168.5.3 tcp dport 53 counter packets 0 bytes 0 dnat to 192.168.5.2:34697
        }

        chain PREROUTING {
                type nat hook prerouting priority dstnat; policy accept;
                counter packets 8 bytes 571 jump LIMADNS
        }

        chain OUTPUT {
                type nat hook output priority -100; policy accept;
                counter packets 11 bytes 726 jump LIMADNS
        }

        chain postrouting {
                type nat hook postrouting priority srcnat; policy accept;
        }
}
$
```

## なんで作ったのか

仕事もプライベートも Debian GNU/Linux sid と Kernel-based Virtual Machine (KVM) を開発環境として利用しています。一時的に利用したいネットワークインタフェースの作成や IP Masquerade の設定を ip コマンドや nft コマンドを利用した書き捨てのシェルスクリプトで作成していましたが、もっと再利用性の高い手段で管理したいなと考えて作ったものになります。設定の永続化はしないため、よくわからなくなったら開発環境を再起動すればよいくらいの雑な使い方をしつつ、シェルスクリプトを書き捨てるよりは再利用性があるを目指して作りました。 dummy interface と bridge interface にしか対応していませんが、いまのところこれだけで自分のやりたいことは十分できています。

余談ですが最初の実装では nftables の操作に [google/nftables](https://github.com/google/nftables) のライブラリを使っていましたが、 comment は Type-length-value format で表現する必要があったりと nftables の API を意識したコードを書かなければならず、今回は nftables のライブラリを必要とするほどの込み入った実装にはなっていないため nft コマンドを利用した実装に変更しました。
