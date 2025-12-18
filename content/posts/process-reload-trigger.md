---
title: "アプリケーションのリロードを補助するツールを作ってみた"
date: 2025-12-14T10:24:59+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 13日目の記事です。

アプリケーションのリロードを補助するツールを作ってみました。

[zinrai/pokego](https://github.com/zinrai/pokego)

## 使い方

[zinrai/pokego](https://github.com/zinrai/pokego) のバイナリをダウンロードしパスのった場所に配置します。

[zinrai/sigcat](https://github.com/zinrai/sigcat) に SIGHUP を送ってみます。

```
$ pokego sighup -name=sigcat
2025/12/18 10:29:08 Successfully poked process sigcat (PID: 1417306) with SIGHUP
$
```

```
$ ./sigcat config.txt
2025/12/18 10:28:36 Loading config from: config.txt
2025/12/18 10:28:36 Config loaded successfully (14 bytes)
=== Config Content ===
Hello, World!
=== End of Config ===
2025/12/18 10:28:36 Daemon started. Send SIGHUP to reload config, SIGINT/SIGTERM to quit.
2025/12/18 10:29:08 Received SIGHUP, reloading config...
2025/12/18 10:29:08 Loading config from: config.txt
2025/12/18 10:29:08 Config loaded successfully (14 bytes)
=== Config Content ===
Hello, World!
=== End of Config ===

```

Prometheus の Exporter などで見掛けるリドードエンドポイントへのリクエストにも対応しています。

```
$ pokego http -url=http://localhost:8080/-/reload
```

## なんで作ったのか

ポ○ Go ではありません。 poke go です。

コンテナで起動しているアプリケーションの設定を更新したときに設定を反映するためにリロードしたいはずで、リロードする手段は SIGHUP かリロードエンドポイントだろうと考え、アプリケーションのリロードに特化したツールがあったら同じようなことをするスクリプトを書き捨てるよりも再利用性が高いのではないかと思い作ってみました。

プロセスをつつく Golang 製のツールということで pokego と名付けています。2012年にルームシェアしていたときにシェアメイトが友人を連れてきて、当時は Facebook に poke という機能があり「 poke って何？　」と私がシェアメイトの友人に聞いたところ、つつくようなジェスチャーをして「これが poke だよ」と教えてくれたのがツール名の決め手になりました。
