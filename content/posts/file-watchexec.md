---
title: "ファイルの変更を検知して任意のコマンドを実行するツールを作ってみた"
date: 2025-12-15T08:23:12+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 15日目の記事です。

ファイルの変更を検知して任意のコマンドを実行するツールを作ってみました。

[zinrai/igotifier](https://github.com/zinrai/igotifier)

## 使い方

[zinrai/igotifier](https://github.com/zinrai/igotifier) のバイナリをダウンロードしてパスの通った場所に配置します。

```
$ igotifier -path="README.md" -exec="echo Hello World"
2025/12/18 08:29:27 Watching "README.md" for changes...
2025/12/18 08:29:38 Executing: echo Hello World
2025/12/18 08:29:38 Command executed successfully
```

## なんで作ったのか

[Git リポジトリから特定のファイルを同期するツールを作ってみた]({{< ref "posts/git-file-sync.md" >}}) で Git リポジトリから設定ファイルを取得し [zinrai/igotifier](https://github.com/zinrai/igotifier) が設定ファイルの変更を検知し [アプリケーションのリロードを補助するツールを作ってみた]({{< ref "posts/process-reload-trigger.md" >}}) を呼び出すと、アプリケーションの設定取得から適用まで行うピタゴラスイッチの出来上がりです。
