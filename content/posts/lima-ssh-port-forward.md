---
title: "Lima 管理下の仮想マシンに SSH Port Foward するツールを作ってみた"
date: 2025-12-08T19:51:45+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 8日目の記事です。

[Lima](https://github.com/lima-vm/lima) 管理下の仮想マシンに SSH Port Foward するツールを作ってみました。

[zinrai/lima-forward](https://github.com/zinrai/lima-forward)

## 使い方

ssh コマンドのラッパースクリプトなので ssh のコマンドラインオプションは全て使えます。

```
$ lima-forward.sh -L 8080:127.0.0.1:80 debian
```

## なぜ作ったのか

DEPRECATED となった limactl コマンドの [show-ssh](https://lima-vm.io/docs/reference/limactl_show-ssh/) サブコマンドには以下のように書かれています。

> Instead, use ‘ssh -F ~/.lima/default/ssh.config lima-default’ .

例えば Lima 管理下の debian 仮想マシンに ssh port forward するために Lima での作法を思い出す必要があり ssh のコマンドラインを組み立てるのが私は辛いなと思いました。 Lima の仮想マシンに ssh 接続するための作法を吸収し「 ssh port forward する」ためのコマンドラインオプションの組み立てだけに集中できる仕組みが欲しいなと思い作りました。
