---
title: "Lima 管理下の複数台の仮想マシンを一括で操作するためのツールを作ってみた"
date: 2025-12-06T16:04:14+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 6日目の記事です。

[Lima](https://github.com/lima-vm/lima) 管理下の複数台の仮想マシンを一括で操作するためのツールを作ってみました。

[lima-compose](https://github.com/zinrai/lima-compose)

## 使い方

[limactl](https://github.com/lima-vm/lima) コマンドがセットアップされている前提で使い方について説明します。

### 設定ファイルの作成

YAML で設定ファイルを書いて設定ファイル内に書かれている仮想マシンを一括操作します。仮想マシンごとに limactl コマンドのコマンドラインオプションを列挙します。

```
cat << EOF > lima-compose.yaml
instances:
  consul-server-01:
    template: template://ubuntu-lts
    args: |
      --cpus 2
      --memory 4
      --disk 50
      --set '.env.NODE_TYPE="server"'
      
  consul-server-02:
    template: template://ubuntu-lts
    args: |
      --cpus 2
      --memory 4
      --disk 50
      --set '.env.NODE_TYPE="server"'
      
  consul-client-01:
    template: template://ubuntu-lts
    args: |
      --cpus 4
      --memory 8
      --disk 100
      --mount ~/projects:/projects:w
      --set '.env.NODE_TYPE="client"'
EOF
```

### 仮想マシンの作成

```
$ lima-compose create lima-compose.yaml
```

### 仮想マシンの起動

```
$ lima-compose start lima-compose.yaml
```

### 仮想マシンの停止

```
$ lima-compose stop lima-compose.yaml
```

### 仮想マシンの削除

```
$ lima-compose destroy lima-compose.yaml
```

## なぜ作ったのか

[Consul](https://developer.hashicorp.com/consul) や [Nomad](https://developer.hashicorp.com/nomad) などクラウタを構成するミドルウェアを Ansible で構成管理していて Lima を使い Ansible Playbook を開発しようとしたときに limactl コマンドを何度も叩かなければならないのが辛く、書き捨てのシェルスクリプトよりは再利用性があり一貫した Lima 管理下の仮想マシンへの操作が可能な手段が欲しいなと考え作りました。 limactl コマンドのコマンドラインオプションを YAML でマッピングすると limactl コマンドのオプションの増減に対応しなければならずコストが高いと考え、 template の指定以外は limactl コマンドのコマンドラインオプションをそのまま記述するという形をとっています。
