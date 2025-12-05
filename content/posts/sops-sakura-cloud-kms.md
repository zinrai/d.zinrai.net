---
title: "sops で「さくらのクラウド KMS」を使うためのツールを作ってみた"
date: 2025-12-02T14:42:44+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 2日目の記事です。

[getsops/sops](https://github.com/getsops/sops) で [さくらのクラウド KMS](https://cloud.sakura.ad.jp/products/kms/)  を使うためのツールを作ってみました。

[zinrai/sakura-kms-sops](https://github.com/zinrai/sakura-kms-sops)

## セットアップ

1. https://github.com/getsops/sops のバイナリをダウンロードしパスの通った場所に配置します
2. https://github.com/zinrai/sakura-kms のバイナリをダウンロードしパスの通った場所に配置します
3. https://github.com/zinrai/sakura-kms-sops の sakura-kms-sops.sh をパスの通った場所に配置します
4. age をインストールします
    - Debian GNU/Linux だと apt でインストールできます 
    - https://packages.debian.org/sid/age 
5. age 鍵を作成します

```
$ age-keygen -o age-key.txt
Public key: age1e86qjmrs6mcw523kuxg72cnfxvpllsfj7ar3s8gkpkq3wvhure8swajalw
```

6. sops の設定ファイルを作成します

$ cat << EOF > .sops.yaml
creation_rules:
  - age: age1e86qjmrs6mcw523kuxg72cnfxvpllsfj7ar3s8gkpkq3wvhure8swajalw
EOF
$ 

7. sakura-kms で age 鍵を暗号化します

```
$ cat age-key.txt | sakura-kms encrypt -output age-key.kms.enc
Successfully encrypted to age-key.kms.enc
```

8. 環境変数を設定します

```
# 作成した API キーのクレデンシャルを設定する
$ export SAKURACLOUD_ACCESS_TOKEN="your-api-token"
$ export SAKURACLOUD_ACCESS_TOKEN_SECRET="your-api-secret"

# 作成した KMS キーのリソース ID を設定する
$ export SAKURACLOUD_KMS_KEY_ID="110000000000"

# さくらのクラウド KMS で暗号化した age 鍵を設定する
$ export SAKURACLOUD_KMS_KEY_FILE="age-key.kms.enc"
```

## 使い方

sops のラッパーコマンドのため sops のオプションやサブコマンドは全て利用できます。

### YAML ファイルの暗号化 

```
$ cat example.yaml
credentials:
  api_key: pgw_live_3fa92f9b1c4d4e8a9cc1d12f5e7b0a91
  webhook_secret: whsec_29b1eabf54c4477c96b0d1f1fe892c3b
```

```
$ sakura-kms-sops.sh -e example.yaml > secret.yaml
```

```
$ cat secret.yaml
credentials:
    api_key: ENC[AES256_GCM,data:Ik9gdmWaK/6SGqEYpR6lvBvsYenk/o030ZwqerxnPL4N/i+jsXdHCzw=,iv:A095+Fhaf3BmOPvWSmd477cT//SSbG77i0cJTygC6yA=,tag:oNxrpn8odR9Gq4nAov8/Sg==,type:str]
    webhook_secret: ENC[AES256_GCM,data:SBcPoAGjAqPTNaoi2WEjpAARSMvIIziCQTo1ekYTVPnTcLTn7ds=,iv:2qHaXFOqOXC16BGyOEgtLC8sZJTXbrWyd1HMA/jBXGg=,tag:iCeTNCs4YbEhX5i6m0sXNQ==,type:str]
sops:
    age:
        - recipient: age1e86qjmrs6mcw523kuxg72cnfxvpllsfj7ar3s8gkpkq3wvhure8swajalw
          enc: |
            -----BEGIN AGE ENCRYPTED FILE-----
            YWdlLWVuY3J5cHRpb24ub3JnL3YxCi0+IFgyNTUxOSBiMUt5bTNQeG5yN1pHK3B5
            cGtMLzgwaWtQSlNwcXRHZ1FEZzdiRkJ2c2xJCjI1bEc0WmRJYXpZa2h0ejZ2QW1Q
            Smx6Y3pRUTIweHVUV2xGanBnN0g5RE0KLS0tIGNzUDlidWt5UisxVkFZNURsc2Fj
            RHFHbEJhRHNHZG5TTkNGZ3VNbVhvVFEKk+jCK86mhYVpSetVM8uc32YLYs9wgtWl
            S1Vw4f/pORO+KKz5hVjyjPuX3COgnADChx6ft2Y91ZsmhASOgIYJeA==
            -----END AGE ENCRYPTED FILE-----
    lastmodified: "2025-12-05T07:13:49Z"
    mac: ENC[AES256_GCM,data:Fd1yk6bmNQ3G/Ib55iw2hD4titPBlbb2XXUeHawndEvNQ9lfWgVTC0Mzh9QSIWPPO6w7wWmBY3c8ncV9nLbMAPWZJzpJfeK9vENmAJ5YAlg1wcFESJQ+mKgjmlCsK+d5cGhOexWkE9HLMyM60IkYvnyUn9cxeYYS8VTVxpiN/E=,iv:4kqltZxuMUWArHfmXyCMY6kut6IWOgc9gWOuAgRVRsc=,tag:9tOt1VBZX2cGB6Y9MqTmOw==,type:str]
    unencrypted_suffix: _unencrypted
    version: 3.11.0
$
```

### YAML ファイルの編集

```
$ sakura-kms-sops.sh edit secret.yaml
credentials:
    api_key: pgw_live_3fa92f9b1c4d4e8a9cc1d12f5e7b0a91
    webhook_secret: whsec_29b1eabf54c4477c96b0d1f1fe892c3b
```

## 仕組み

[getsops/sops](https://github.com/getsops/sops) は [AWS KMS](https://aws.amazon.com/jp/kms/) , [Azure Key Vault](https://learn.microsoft.com/ja-jp/azure/key-vault/general/basic-concepts) , [Cloud KMS](https://cloud.google.com/security/products/security-key-management?hl=ja) など主要なクラウドプロバイダの KMS には対応していますが、 さくらのクラウド KMS には対応していません。それでもどうにかして使う方法はないだろうかと sops の README.md を眺めていたところ age 鍵を sops で使うことができ `SOPS_AGE_KEY_CMD` 環境変数で age 鍵を呼び出すコマンドラインを書けそうな記述が [2.3 Encrypting using age](https://github.com/getsops/sops?tab=readme-ov-file#encrypting-using-age) に書かれていました。

さくらのクラウド KMS を直接利用するパスはないけれど、 age 鍵を「さくらのクラウド KMS 」で暗号化して `SOPS_AGE_KEY_CMD` に「さくらのクラウド KMS 」で age 鍵を復号化するコマンドラインを記述すれば sops で「さくらのクラウド KMS 」を使えるのではないかという発想を具体化したものになります。

## なんで作ったのか

業務で sops を使って機密情報を管理するオペレーションを構築したいなと考えていて、「さくらのクラウド」で KMS が提供されているので、これを使って sops を使えないか模索した結果、今回のツールが出来上がりました。

私が [zinrai/sakura-kms-sops](https://github.com/zinrai/sakura-kms-sops) を作った5日後くらいに [fujiwara/sops-sakura-kms](https://github.com/fujiwara/sops-sakura-kms) が公開され [HashiCorp Vault Transit secrets engine](https://developer.hashicorp.com/vault/api-docs/secret/transit) の互換 HTTP サーバーを動かせば age 鍵も必要ないし、言われてみればそうだけど sops の README.md を眺めたときには思い付かなかったというお気持ちになりました。

[fujiwara/sops-sakura-kms](https://github.com/fujiwara/sops-sakura-kms) のほうが使い勝手がよいと思いますが、どのように考えて実装したのかを記録として残しておきます。
