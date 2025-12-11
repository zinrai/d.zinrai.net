---
title: "Grafana Loki の LogCLI をバックエンドにしたクエリビルダーを作ってみた"
date: 2025-12-10T08:55:54+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 10日目の記事です。

Grafana Loki の [LogCLI](https://grafana.com/docs/loki/latest/query/logcli/) をバックエンドにしたクエリビルダーを作ってみました。

[zinrai/loqui](https://github.com/zinrai/loqui)

## 事前準備

* [logcli](https://github.com/grafana/loki) のバイナリをダウンロードしてパスの通った場所に配置します
* [fzf](https://github.com/junegunn/fzf) のバイナリをダウンロードしてパスの通った場所に配置します
* [zinrai/loqui](https://github.com/zinrai/loqui) のバイナリをダウンロードしてパスの通った場所に置します

## 使い方

logcli のコマンドラインをインタラクティブに組み立てるだけの場合

```
$ export LOKI_ADDR=http://localhost:3100
$ loqui

Select time range type:
1. Relative (e.g., 1h, 24h)
2. Absolute (specific dates)
Enter choice (1-2): 2

Enter start time (YYYY-MM-DD HH:MM or YYYY-MM-DD): 2025-08-14 09:00
Enter end time (YYYY-MM-DD HH:MM or YYYY-MM-DD): 2025-08-14 18:00

# Interactive fzf selection of labels
Select label: app

Select operator for 'app' (default: 1):
1. = (equals)
2. != (not equals)
3. =~ (regex match)
4. !~ (regex not match)
Enter number (1-4) or press Enter for default: 1

# Interactive fzf selection of values
Select value for 'app': nginx

=== Current labels ===
[SET] app="nginx"

Add more labels? (y/N): y

Select label: env
Select value for 'env': production

=== Current labels ===
[SET] app="nginx"
[SET] env="production"

Add more labels? (y/N): [Enter]

Add line filter? (y/N): y

Select line filter operator (default: 1):
1. |= (contains)
2. != (does not contain)
3. |~ (matches regex)
4. !~ (does not match regex)
Enter number (1-4) or press Enter for default: 1

Enter filter text: error

# Output:
logcli query '{app="nginx",env="production"} |= "error"' --from 2025-08-14T09:00:00+09:00 --to 2025-08-14T18:00:00+09:00
```

logcli のコマンドラインを組み立てて実行する場合

```
$ loqui -exec
```

## 仕組み

インタラクティブな操作の後ろで logcli コマンドが実行され label や label の値が fzf に渡されて絞り込みながら選択できるようになっています。

## なんで作ったのか

ロクイと読みます。

チームメンバー内で Loki のログ検索のクエリを渡し合うときに Grafana のリンクの渡し合うよりも logcli のコマンドライン渡すほうが再現性や再利用性が高いのではないかと考えています。

ただし logcli だけだと対象の日時の label を絞り込み、その label の値を絞り込み LogQL を書くという一連の流れにおいて何度も logcli コマンドを叩かなければならず本当に辛くて使う気になれません。また対象の日時を指定する --from と --to を RFC3339 で書かなければならないというのも認知負荷が高すぎではないかと思っています。

逆に言えば Grafana のクエリビルダーのように、これらを補助するツールを被せればチームメンバー内の logcli のコマンドラインの渡し合いを実現できるのではないかと作ってみました。
