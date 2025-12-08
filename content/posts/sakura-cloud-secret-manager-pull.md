---
title: "「さくらのクラウド シークレットマネージャ」からシークレット情報を取得するツールを作ってみた"
date: 2025-12-04T11:33:55+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 3 5日目の記事です。

[さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) からシークレット情報を取得するツールを作ってみました。

[zinrai/sakura-secrets-pull](https://github.com/zinrai/sakura-secrets-pull)

## 使い方

[シークレットマネージャ](https://manual.sakura.ad.jp/cloud/appliance/secretsmanager/index.html)  のドキュメントより「シークレット保管庫」を作成済みかつ、 [APIキー](https://manual.sakura.ad.jp/cloud/api/apikey.html) のドキュメントより「さくらのクラウド シークレットマネージャ」を利用するための API キーを作成済みであることを前提に使い方を説明していきます。

### 環境変数の設定

```
# 作成した API キーのクレデンシャルを設定する
$ export SAKURACLOUD_ACCESS_TOKEN="your-access-token"
$ export SAKURACLOUD_ACCESS_TOKEN_SECRET="your-access-token-secret"
```

### 設定ファイルの作成

対象の「シークレット保管庫」に登録されているシークレット情報を指定して dest には書き出すパスとファイル名を設定します。 ( id には作成した「シークレット保管庫」のリソース ID を設定します。 )

```
$ cat << EOF > secrets-config.yaml
vault:
  id: "123456789012"  # Your vault resource ID

secrets:
  - name: production-db-password
    dest: db_password
EOF
```

### シークレット情報の取得

設定に従い「シークレット保管庫」からシークレット情報を取得しファイルに書き出します。

```
$ sakura-secrets-pull -config secrets-config.yaml
```

## なんで作ったのか

「さくらのクラウド シークレットマネージャ」には SSH 秘密鍵やシークレット情報などを含んだ設定ファイルなどが登録されるはずで、シークレット情報をファイルとして取得して利用するケースがあるだろうと考え、これを実現するためのツールが存在しなかったため作りました。
