---
title: "Alertmanager を起動せずに Routing を検証するツールを作ってみた"
date: 2025-12-12T23:04:41+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 12日目の記事です。

[Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) を起動せずに Alertmanager の [Routing](https://prometheus.io/docs/alerting/latest/configuration/#route-related-settings) を検証するツールを作ってみました。

[zinrai/amroutify](https://github.com/zinrai/amroutify)

## 使い方

[zinrai/amroutify](https://github.com/zinrai/amroutify) のバイナリをダウンロードしてパスの通った場所に配置します。

リポジトリ内に example を用意していて、どんな感じになるかを試せるようになっています。

```
$ amroutify -config example/alertmanager.yml -tests example/routing_tests.yml
PASS: Maintenance mode - alerts discarded
PASS: Batch job alert - scheduler webhook only
PASS: Batch job with severity info
PASS: Ino severity - dedicated channel
PASS: Debug alert - firing only
PASS: Debug alert - with recovery
PASS: Warning alert - firing
PASS: Warning alert - recovery
PASS: Critical alert - firing (multi-receiver)
PASS: Critical alert - recovery (multi-receiver)
PASS: Critical alert - phone escalation for platform team
PASS: Critical alert - phone escalation with recovery
PASS: Batch job with critical severity - firing
PASS: Batch job with warning severity
PASS: Unknown severity - falls through to default
PASS: No severity label - default receiver
PASS: Maintenance overrides critical
PASS: Empty batch_job_id does not match regex

Results: 18 passed, 0 failed, 18 total
$
```

## なんで作ったのか

現実の Alertmanager の Routing 設定はサンプルよりもずっと複雑であると感じています。

「 Critical は内製のシステムに通知したい」「全ての通知は Slack に通知したい」「 A ラベルがあるときはワークフローエンジンに通知したい」「 B ラベルがあるときはメールも出したい」「 C ラベルがあるときは復旧を通知したい」といった要件に答えようとすると continue だらけの複雑な Routing 設定の出来上がりです。

さらに設定を足したくなった場合、追加した設定が意図したルーティングになっているか、設定を追加したことによって既存のルーティングがデグレーションしていないかといったことを確認する作業が発生します。これが手動だと非常にトイルで、作業が発生したときにはとても憂鬱な気持ちです。

Alertmanager の Routing はラベルを利用してルーティングを制御するため、「あるラベルの入力があったときに返ってくる receiver はこれ」といった期待値をテーブル駆動テスト的な感じで記述し、 Alertmanager の設定と突き合わせるというテストが Alertmanager を起動せずにできたら手動での憂鬱な作業から開放されるのではないかと考えました。

この課題感と解いているツールが OSS で公開されていたりするのか調べてみましたが見当らなかったため、 Alertmanager は Golang で書かれていて Alertmanager の YAML 設定のロードと Routing の評価は Alertmanager の API を import すればやりたいことがコードをそんなに書かずに実現できるのではないかと考え作ってみました。
