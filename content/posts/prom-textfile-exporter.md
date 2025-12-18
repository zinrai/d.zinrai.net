---
title: "Node exporter の Textfile Collector 用メトリクスを YAML で定義して生成するツールを作ってみた"
date: 2025-12-17T13:11:26+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 17日目の記事です。

Node exporter の Textfile Collector 用メトリクスを YAML で定義して生成するツールを作ってみました。

[zinrai/prom-textfile-exporter](https://github.com/zinrai/prom-textfile-exporter)

Node exporter の [Textfile Collector](https://github.com/prometheus/node_exporter?tab=readme-ov-file#textfile-collector) の機能を利用します。 YAML でメトリクスを定義し、コマンドのリターンコードをメトリクスの値にしたり、コマンドのリターンコードを別の値にマッピングしてメトリクスの値としたり、コマンドの出力結果から数値を取り出してメトリクスの値にする機能を提供しています。

## 使い方


```
$ cat examples/dns_check.yaml
metrics:
  # Check if DNS resolution works for multiple domains
  dns_resolution:
    name: "dns_resolution_status"
    type: "gauge"
    help: "DNS resolution status (1=success, 0=failure)"
    collector:
      type: "returncode_mapping" # メトリクスの値を別の値にマッピングしてメトリクスの値とします
      command: "nslookup google.com"
      mapping:
        "0": 1
        "default": 0
      labels:
        domain: "google.com"
        type: "public"

  dns_resolution_internal:
    name: "dns_resolution_status"
    type: "gauge"
    help: "DNS resolution status (1=success, 0=failure)"
    collector:
      type: "returncode" # リターンコードの値をそのままメトリクスの値とします
      command: "nslookup internal.example.com"
      labels:
        domain: "internal.example.com"
        type: "internal"
```

-output-dir オプションを設定すると指定のディレクトリにメトリクスのファイルを生成します。

```
$ prom-textfile-exporter run -config examples/dns_check.yaml -output-dir /tmp
2025/12/18 13:43:06 Loading configuration from: examples/dns_check.yaml
2025/12/18 13:43:06 Collecting metric: dns_resolution
2025/12/18 13:43:06 Collecting metric: dns_resolution_internal
2025/12/18 13:43:07 Successfully wrote 2 metrics to /tmp/prom_textfile_exporter.prom
$
```

```
$ cat /tmp/prom_textfile_exporter.prom
# HELP dns_resolution_status DNS resolution status (1=success, 0=failure)
# TYPE dns_resolution_status gauge
dns_resolution_status{domain="google.com",type="public"} 1
dns_resolution_status{domain="internal.example.com",type="internal"} 0
$
```

-output-dir オプションを設定していない場合は生成されるメトリクスを標準出力します。

```
$ prom-textfile-exporter run -config examples/dns_check.yaml
2025/12/18 13:43:00 Loading configuration from: examples/dns_check.yaml
2025/12/18 13:43:00 Collecting metric: dns_resolution
2025/12/18 13:43:00 Collecting metric: dns_resolution_internal
# HELP dns_resolution_status DNS resolution status (1=success, 0=failure)
# TYPE dns_resolution_status gauge
dns_resolution_status{domain="google.com",type="public"} 1
dns_resolution_status{type="internal",domain="internal.example.com"} 0
```

## なんで作ったのか

Node exporter の [Textfile Collector](https://github.com/prometheus/node_exporter?tab=readme-ov-file#textfile-collector)  を利用する機会がたまにあります。コマンドがリターンコードで状態を表現していて、コマンドのリターンコードを Textfile Collector 経由でメトリクスの値として取得したいという状況がたまにあります。シェルスクリプトを作成して Textfile Collector 用のメトリクスを生成しているのではないでしょうか。

メトリクスを生成するためのシェルスクリプトを個別に作成するよりも、やりたいことは「コマンドのリターンコードを Prometheus 形式のメトリクスの値として利用する」ことだと考えていて、それなら統一された書式で Textfile Collector 用のメトリクスを生成するツールがあったほうが再利用性が高いのではないかと思い作ってみました。
