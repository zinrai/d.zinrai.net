---
title: "systemd timer のオペレーションを一括操作するツールを作ってみた"
date: 2025-12-16T11:18:39+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 3 16日目の記事です。

systemd timer のオペレーションを一括操作するツールを作ってみました。

[zinrai/systemd-timer-operator](https://github.com/zinrai/systemd-timer-operator)

## 使い方

[zinrai/systemd-timer-operator](https://github.com/zinrai/systemd-timer-operator) のバイナリをダウンロードしてパスの通った場所に配置します。

service unit と timer unit の設定は YAML で管理します。

```
$ cat examples/basic.yml
Unit: |
  Description=Hello World Timer Example

Service: |
  Type=oneshot
  ExecStart=/usr/bin/echo "Hello from systemd-timer-operator!"

Timer: |
  OnCalendar=minutely

TimerInstall: |
  WantedBy=timers.target
$
```

enable サブコマンドに systemd timer 用の YAML 設定を指定すると service unit , timer unit が配置され systemd timer が登録されます。

```
$ sudo systemd-timer-operator enable examples/basic.yml
Created: /etc/systemd/system/systemd-timer-operator-basic.service
Created: /etc/systemd/system/systemd-timer-operator-basic.timer
Executed: systemctl daemon-reload
Created symlink /etc/systemd/system/timers.target.wants/systemd-timer-operator-basic.timer → /etc/systemd/system/systemd-timer-operator-basic.timer.
Executed: systemctl enable --now systemd-timer-operator-basic.timer
$
```

```
$ systemctl status systemd-timer-operator-basic
○ systemd-timer-operator-basic.service - Hello World Timer Example
     Loaded: loaded (/etc/systemd/system/systemd-timer-operator-basic.service; static)
     Active: inactive (dead)
TriggeredBy: ● systemd-timer-operator-basic.timer
$
```

disable サブコマンドに systemd timer 用の YAML 設定を指定すると systemd timer の登録が削除され service unit も削除されます。

```
$ sudo systemd-timer-operator disable examples/basic.yml
Removed "/etc/systemd/system/timers.target.wants/systemd-timer-operator-basic.timer".
Executed: systemctl disable --now systemd-timer-operator-basic.timer
Removed: /etc/systemd/system/systemd-timer-operator-basic.timer
Removed: /etc/systemd/system/systemd-timer-operator-basic.service
Executed: systemctl daemon-reload
$
```

```
$ systemctl status systemd-timer-operator-basic
Unit systemd-timer-operator-basic.service could not be found.
$ 
```

service unit と timer unit を生成するだけの generate サブコマンドも提供しています。

```
$ systemd-timer-operator generate examples/basic.yml Generated: systemd-timer-operator-basic.service
Generated: systemd-timer-operator-basic.timer
$
```

```
$ cat systemd-timer-operator-basic.service
[Unit]
Description=Hello World Timer Example

[Service]
Type=oneshot
ExecStart=/usr/bin/echo "Hello from systemd-timer-operator!"
$ cat systemd-timer-operator-basic.timer
[Unit]
Description=Hello World Timer Example

[Timer]
OnCalendar=minutely

[Install]
WantedBy=timers.target
$
```

## なんで作ったのか

systemd timer を登録する一連のオペレーションです。

1. service unit を作成する
2. timer unit を作成する
3. service unit と timer unit を配置する
4. systemctl daemon-reload を実行する
5. systemctl enable --now timer-name.timer を実行する

辛いです。

最近、個人的に利用している VPS ( さくらの VPS ではない ) にて cron を使おうと/etc/cron.d に cron の設定を配置したところ動かず、調べてみると [systemd-cron](https://packages.debian.org/sid/systemd-cron) というパッケージをインストールして cron を使うという手段があることを知りました。また cron がインストールされていないという環境が存在することもはじめて体験しました。 systemd timer を強いられる未来があるかはわかりませんが systemd timer を強いられる未来があったときのために作ってみました。( そして私は apt install cron するのでした。 )
