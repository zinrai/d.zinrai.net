---
title: "システム連携で対象サーバーのコマンドを実行したいときに SSH を使わないツールを作ってみた"
date: 2025-12-23T03:28:21+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 23日目の記事です。

システム連携で対象サーバーのコマンドを実行したいときに SSH を使わないツールを作ってみました。

[zinrai/savalet](https://github.com/zinrai/savalet)

HTTP API 経由でサーバーにあるコマンドを実行して結果を返すツールになります。コマンドを実行するデーモンと実行するコマンドのリクエストを受け付ける HTTP API サーバーで構成されています。コマンドを実行するデーモンと HTTP API サーバー間は unix domain socket で通信します。コマンドを実行するデーモン側で実行を許可するコマンドを設定します。

サバレットと読んでください。「サーバーの従者」という意味を込め Server Valet を略して savalet という名前にしました。 servalet だと Java Servlet  に似ていて sevalet だとサバレットとは読むのは苦しいため sa をローマ字読みにして valet でサバレットとしました。サバがサーバーを表現できてそうで個人的には気に入っています。

savalet 自体には認証機能はありません。サーバーのコマンドを実行できる強力な機能のため savalet の手前に認証機構を入れることを推奨します。

詳細なアーキテクチャは [zinrai/savalet](https://github.com/zinrai/savalet) の README を参照してください。

## 使い方

コマンドを実行するデーモンを起動します。

```
$ ./savalet daemon -config example/daemon.yaml
2025/12/25 03:24:39 Starting savalet daemon (version: 0.1.0)
2025/12/25 03:24:39 Socket path: /tmp/savalet.sock
2025/12/25 03:24:39 Daemon listening on /tmp/savalet.sock
2025/12/25 03:24:56 {"timestamp":"2025-12-24T18:24:56Z","level":"info","mode":"daemon","event":"command_executed","command":"uptime","execution_time":"1.369603ms"}
```

HTTP API サーバーを起動します。

```
$ ./savalet api -config example/api.yaml
2025/12/25 03:24:44 Starting savalet API server (version: 0.1.0)
2025/12/25 03:24:44 Listen address: :9090
2025/12/25 03:24:44 Socket path: /tmp/savalet.sock
2025/12/25 03:24:44 Request timeout: 60 seconds
2025/12/25 03:24:44 Successfully connected to daemon at /tmp/savalet.sock
2025/12/25 03:24:44 API server listening on :9090
2025/12/25 03:24:56 {"timestamp":"2025-12-24T18:24:56Z","level":"info","mode":"api","event":"http_request","method":"POST","path":"/execute","remote_addr":"[::1]:33412","status":200,"latency":"2.124777ms"}
```

HTTP API 経由でコマンドを実行します。

```
$ curl -X POST http://localhost:9090/execute -d '{"command":"uptime","args":[],"timeout":10}'
{"success":true,"stdout":"03:24:56 up 10 days,  5:35,  2 users,  load average: 0.09, 0.16, 0.11","execution_time":"1.369603ms"}
```

## なんで作ったのか

システム連携で対象のサーバーのコマンドを実行したくなったときに SSH を使ってサーバーのコマンドを実行する実装をすることがあるはずです。10年以上インフラエンジニアをやってきた所感としては「あのディスクが壊れた物理サーバーの中にしか SSH 鍵入ってなかったのか」「 fingerprint 取り込んでいなくてバッチ止まってるよ」「退職した A さんの手元にあった SSH 鍵で動いていたのか」「バージョンアップで SSH v1 クライアントからは接続できなくなるんだって」「連携元が SSH v1 だけど連携先をバージョンアップすると SSH v2 になるので連携元に SSH v2 のクライアントをビルドしてインストールしないと」などなど実装は簡単だけど禍根を残しやすい構成だなと感じています。

対象のサーバーで実行したいコマンドは限られているはずで、実行するコマンドの許可リストを設定して許可リストにマッチしたコマンドのみを実行する機能と HTTP API 経由でコマンドの実行をリクエストできたら SSH 呼び出しした先に待ち受けている未来を回避できるのではないかと考えました。 HTTP API での連携が可能になるためワークフローエンジンや CI/CD ツールとの連携がしやすくなるのではないかと思っています。
