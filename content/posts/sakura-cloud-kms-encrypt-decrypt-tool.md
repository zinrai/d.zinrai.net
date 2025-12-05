---
title: "「さくらのクラウド KMS」を使ってファイルを暗号化・復号化するツールを作ってみた"
date: 2025-12-01T12:54:47+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 1日目の記事です。

Key Management Service である [さくらのクラウド KMS](https://clud.sakura.ad.jp/products/kms/) の管理下にある鍵を使いファイルを暗号化、復号化するツールを作ってみました。

[zinrai/sakura-kms](https://github.com/zinrai/sakura-kms)

# 使い方

[KMS（Key Management Service）](https://manual.sakura.ad.jp/cloud/appliance/kms/index.html)  のドキュメントより「 KMS キー」を作成済みかつ、 [APIキー](https://manual.sakura.ad.jp/cloud/api/apikey.html) のドキュメントより「さくらのクラウド KMS 」を利用するための API キーを作成済みであることを前提に使い方を説明していきます。

## バイナリのダウンロード

[zinrai/sakura-kms](https://github.com/zinrai/sakura-kms) からバイナリをダウンロードしてください。

## 環境変数の設定

```
# 作成した API キーのクレデンシャルを設定する
$ export SAKURACLOUD_ACCESS_TOKEN="your-api-token"
$ export SAKURACLOUD_ACCESS_TOKEN_SECRET="your-api-secret"

# 作成した KMS キーのリソース ID を設定する
$ export SAKURACLOUD_KMS_KEY_ID="110000000000"
```

## ファイルの暗号化

```
$ cat hello_world.txt
Hello World
$ cat hello_world.txt | sakura-kms encrypt -output hello_world.txt.enc
Successfully encrypted to hello_world.txt.enc
$ cat hello_world.txt.enc
g6NhbGerYWVzLTI1Ni1nY22ja2V5g6JpZKwxMTM3MDI0NDU1MTeia3YAo2NpcNl3c29zOnYxOmc2UmliRzlpeENCbGJENU9IUVVRTkFiekYzWTZKdGlTSm5rcmNtRHcrWkhia2graFdaQWxGYUpwZHNRUTBMbnFlY3Y1VHVXR2RHaFlFTHZZSWFOMFlXZkVFTDJFTWFqOHFTZXBrT05zUWVWZEwzND2jdmFsxDx5sT70m0LXxMSjWrbvRrgnSNVULM04083czcD09NO9yg8fcl0otQvFtsW0CDq0udBqcyqjdNxiUF8hj/0=
$
```

## ファイルの復号化

```
$ cat hello_world.txt.enc
g6NhbGerYWVzLTI1Ni1nY22ja2V5g6JpZKwxMTM3MDI0NDU1MTeia3YAo2NpcNl3c29zOnYxOmc2UmliRzlpeENCbGJENU9IUVVRTkFiekYzWTZKdGlTSm5rcmNtRHcrWkhia2graFdaQWxGYUpwZHNRUTBMbnFlY3Y1VHVXR2RHaFlFTHZZSWFOMFlXZkVFTDJFTWFqOHFTZXBrT05zUWVWZEwzND2jdmFsxDx5sT70m0LXxMSjWrbvRrgnSNVULM04083czcD09NO9yg8fcl0otQvFtsW0CDq0udBqcyqjdNxiUF8hj/0=
$ cat hello_world.txt.enc | sakura-kms decrypt -output hello_world.txt.dec
Successfully decrypted to hello_world.txt.dec
$ cat hello_world.txt.dec
Hello World
$
```

# なんで作ったのか

さくらのクラウド KMS は [さくらのクラウド シークレットマネージャ](https://cloud.sakura.ad.jp/products/secrets-manager/) などで利用する鍵として使う想定のもので、さらのクラウド KMS を直接利用するというユースケースはなかなか無いと思われます。今回は、 さくらのクラウド KMS の管理下にある鍵を使って [getops/sops](https://github.com/getsops/sops) を使いたくて作りました。

今回のツールを組み合わせて、さくらのクラウド KMS と [getops/sops](https://github.com/getsops/sops) を連携させるツールを作った話は [さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 2日目に公開します。
