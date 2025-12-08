---
title: "Lima 管理下の仮想マシンに Ansible Playook を適用するための Ansible Dynamic Inventory を作ってみた"
date: 2025-12-07T16:39:04+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 7日目の記事です。

[Lima](https://github.com/lima-vm/lima) 管理下の仮想マシンに Ansible Playbook を適用するための Ansible Dynamic Inventory を作ってみました。

[zinrai/limventory](https://github.com/zinrai/limventory)

## 使い方

```
$ ansible-playbook -i ./limventory.py site.yml
```

## 仕組み

limactl list --format json コマンドで Lima 管理下の仮想マシンの情報を取得し Ansible Dynamic Inventory のパラメータを生成しています。 Ansible Playbook を適用する仮想マシンの絞り込みは Lima の env を変換して ansible-playbook コマンドの -l オプションで利用できるようにしています。

## なんで作ったのか

Lima 管理下の仮想マシンで Ansible Playbook を開発しようとしたときに static inventroy ではチームメンバーが開発しようとしたときに、そのメンバーの手元の環境に合わせた static inventory に編集しなければならず Ansible Dynamic Inventory を使う必要がありそうだと考えました。 limactl コマンドのコマンドラインオプションを眺めてみたところ JSON で仮想マシンの情報を取得できることがわかり Ansible Dynamic Inventory を作りました。
