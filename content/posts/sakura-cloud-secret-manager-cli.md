---
title: "「さくらのクラウド シークレットマネージャ」を操作するためのツールを作ってみた"
date: 2025-12-03T22:51:08+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 3日目の記事です。

[さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) を操作するためのツールを作ってみました。

[zinrai/sakura-secrets](https://github.com/zinrai/sakura-secrets)

[さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) は機密情報を保管・管理するサービスです。 [さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) の管理下にある機密情報は [さくらのクラウド KMS](https://cloud.sakura.ad.jp/products/kms/) の鍵を使い暗号化、復号化されます。

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

### シークレット情報の登録

stdin から機密情報を入力するようにしています。

```
$ sakura-secrets put -name secret.yaml < secret.yaml
Successfully created/updated secret: secret.yaml
```

### シークレット情報の削除

```
$ sakura-secrets delete -name secret.yaml
Successfully deleted secret: secret.yaml
```

## なんで作ったのか

[SakuraCloud Provider](https://registry.terraform.io/providers/sacloud/sakuracloud/latest/docs) の [sakuracloud_secret_manager_secret](https://registry.terraform.io/providers/sacloud/sakuracloud/latest/docs/resources/secret_manager_secret) resource を使うか、 [KMS/SecretManager/CloudHSM API ドキュメント](https://manual.sakura.ad.jp/api/cloud/security-encryption/#tag/secretmanager-vault) を参照してツールを自作するかの二択しかなく、 Terraform は tfstate で機密情報が見えそうかつちょっと試してみるには腰が重く、利用者に [KMS/SecretManager/CloudHSM API ドキュメント](https://manual.sakura.ad.jp/api/cloud/security-encryption/#tag/secretmanager-vault) を利用してツールを開発してもらうのはさらに腰が重いだろうということで作ってみました。シークレット情報の参照は熟考の余地がありそうだなと思っていて参照は機能に含めていません。機能としてはシークレット情報の登録、削除、一覧になります。
