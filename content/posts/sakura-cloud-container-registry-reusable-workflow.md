---
title: "「さくらのクラウド コンテナレジストリ」に Docker イメージをアップロードする Reuse workflows を作ってみた"
date: 2025-12-09T07:55:28+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 9日目の記事です。

[さくらのクラウド コンテナレジストリ](https://cloud.sakura.ad.jp/products/container-registry/) に Docker イメージをアップロードする GitHub Actions の Reuse workflows を作ってみました。

[zinrai/actions-sakura-cloud-container-registry](https://github.com/zinrai/actions-sakura-cloud-container-registry)

## 使い方

[さくらのクラウド コンテナレジストリ](https://manual.sakura.ad.jp/cloud/appliance/container-registry/index.html) にて作成したユーザー名とパスワードを GitHub Actions のシークレットに登録します。

[GitHub Actions でのシークレットの使用](https://docs.github.com/ja/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

GitHub Actions のコードをリポジトリに追加します。

```
$ mkdir -p .github/workflows
$ cat << EOF > .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    uses: zinrai/actions-sakura-cloud-registry/.github/workflows/build-push.yml@v1
    with:
      registry-name: your-registry.sakuracr.jp
    secrets:
      registry-username: ${{ secrets.SAKURA_CLOUD_REGISTRY_USERNAME }}
      registry-password: ${{ secrets.SAKURA_CLOUD_REGISTRY_PASSWORD }}
EOF
```

git tag を作成し tag を push します。

```
$ git tag v1.0.0
$ git push origin v1.0.0
```

## なんで作ったのか

2025年12月9日から [さくらのクラウド コンテナレジストリ](https://manual.sakura.ad.jp/cloud/appliance/container-registry/index.html) が正式版の提供を開始しました。 GitHub や他のクラウドサービスもコンテナレジストリを提供しているなかユースケースがないと裾野は広がらないと思い、 GitHub Actions から Docker イメージを作成してアップロードするのはよくあるユースケースではないかと考え [Reuse workflows](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) として公開しました。認知負荷を下げたくて必須のパラメータは少なくして、必要な場合のみ追加のパラメータで調整するという構成をとっています。パラメータの調整が必要な場合はリポジトリの README.md を参照してください。
