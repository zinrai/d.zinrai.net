---
title: "「さくらのクラウド シークレットマネージャ」に登録されているシークレット情報との差分の有無を確認するツールを作ってみた"
date: 2025-12-05T13:10:54+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 3 5日目の記事です。

[さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) に登録されているシークレット情報との差分の有無を確認するツールを作ってみました。

[zinrai/sakura-secrets-diff](https://github.com/zinrai/sakura-secrets-diff)

## 使い方

[シークレットマネージャ](https://manual.sakura.ad.jp/cloud/appliance/secretsmanager/index.html)  のドキュメントより「シークレット保管庫」を作成済みかつ、 [APIキー](https://manual.sakura.ad.jp/cloud/api/apikey.html) のドキュメントより「さくらのクラウド シークレットマネージャ」を利用するための API キーを作成済みであることを前提に使い方を説明していきます。

### 環境変数の設定

```
# 作成した API キーのクレデンシャルを設定する
$ export SAKURACLOUD_ACCESS_TOKEN="your-access-token"
$ export SAKURACLOUD_ACCESS_TOKEN_SECRET="your-access-token-secret"

# 作成したシークレット保管庫のリソース ID を設定する
$ export SAKURACLOUD_SECRETS_ID="your-vault-resource-id"
```

### 差分あり

```
$ printf '%s' "$(cat example.yaml)" | sakura-secrets-diff -name api-token
api-token : With Differences
$
```

### 差分なし

```
$ printf '%s' "$(cat example.yaml)" | sakura-secrets-diff -name secret.yaml
secret.yaml : No Differences
```

## なんで作ったのか

シークレット情報を暗号化してリポジトリで管理し [zinrai/sakura-secrets](https://github.com/zinrai/sakura-secrets) と GitHub Actions を組み合わせて「さくらのクラウド シークレットマネージャ」にシークレット情報を登録する構成を考えたときに、シークレット情報を更新するときのオペレーションにて編集したシークレット情報以外に変更がないことを確認できると安心してシークレット情報の更新ができるのではないかと考え作りました。
