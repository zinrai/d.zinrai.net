---
title: "モノレポにて Git タグ駆動でアプリケーションごとの Docker イメージ作成を実現するツールを作ってみた"
date: 2025-12-25T07:59:26+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 25日目の記事です。

モノレポにて Git タグ駆動でアプリケーションごとの Docker イメージ作成を実現するツールを作ってみました。

[zinrai/monobake](https://github.com/zinrai/monobake)

## 使い方

[zinrai/monobake](https://github.com/zinrai/monobake) のバイナリをダウンロードしてパスの通った場所に配置します。

例えば backend/v1.0.0 の Git タグを渡すと backend と semver を返します。

```
$ monobake -tag refs/tags/backend/v1.0.0
backend 1.0.0
$ 
```

これがモノレポでの Docker イメージ作成と何の関係があるのか説明します。

monobake が返す値を [docker buildx bake](https://docs.docker.com/reference/cli/docker/buildx/bake/) に渡すことでモノレポでのアプリケーションごとの Docker イメージ作成を実現します。

Docker Bake は docker が提供している機能で、複数の Docker イメージのビルド設定をファイルで記述します。設定は HCL , JSON , YAML などで記述できます。以下は JSON での例です。

```
$ cat docker-bake.json
{
  "target": {
    "backend": {
      "context": "apps/backend",
      "dockerfile": "Dockerfile"
    },
    "frontend": {
      "context": "apps/frontend",
      "dockerfile": "Dockerfile"
    },
    "worker": {
      "context": "apps/worker",
      "dockerfile": "Dockerfile"
    }
  },
  "group": {
    "default": {
      "targets": ["backend", "frontend", "worker"]
    }
  }
}
$
```

Docker Bake のコマンドには --set というオプションが提供されており、ビルド設定のファイルに加えて動的にパラメータを上書きすることが可能になっています。 monobake は docker buildx bake に渡すためのパラメータを生成して、 monobake からのパラメータを受け取った Docker Bake は対象アプリケーションの Docker イメージのみを作成します。これによってモノレポにて Git タグ駆動によるアプリケーションごとの Docker イメージ作成を実現します。以下は monobake と Docker Bake を組み合わせた例になります。

```
read TARGET VERSION <<< $(monobake -tag refs/tags/backend/v1.0.0)
[ -n "$TARGET" ] && docker buildx bake \
  --set="${TARGET}.tags=${REGISTRY}/${TARGET}:${VERSION}" \
  "$TARGET" --push
```

monobake と Docker Bake を組み合わせて GitHub Actions を使った Git タグ駆動によるアプリケーションごとの Docker イメージ作成を実現しているデモ用のコードを [zinrai/monobake-demo](https://github.com/zinrai/monobake-demo) にて公開しているので具体的な利用方法をイメージするために参照してください。

## なんで作ったのか

複数のアプリケーションがディレクトリを分けて Dockerfile がそれぞれに配置されている構成のリポジトリがあった場合に、どうやって個々のアプリケーションを Git タグ駆動で Docker イメージにしていくのだろうかと「モノレポ」という単語を含めてググってみたところ Nx、Bazel、Turborepo のようなツールが目に入り「 Docker イメージを作成したいだけなのだけれどモノレポはここまでのことをしないといけないのだろうかハードルが高くないか」と思いました。 GitHub Actions でやろうとするとどうなりそうかもググてみたところ GitHub Actions の YAML 内にシェルスクリプトや jq を駆使してゴリゴリにロジックを書きそうな気配を感じました。

Nx , Bazel , Turborepo のようなツールを学習して導入するのはコストが高いなと思いました。 GitHub Actions の YAML にロジックをゴリゴリに書くと手元で再現しにくくなり、 YAML を修正するたびに「動きますように」と祈らなければならずデバッグも難しくなるためやりたくないなと思いました。

もしかしてこれらの手段の中間にあたる層が空白地帯なのではないか、 GitHub Actions などにロジックをほとんど書かずに実行できて手元の環境でも再現できるようなやり方はないかと調べて monobake を作って Docker Bake と組み合わせるという結論に至りました。

[zinrai/monobake-demo](https://github.com/zinrai/monobake-demo) にあるように GitHub Actions にほとんどロジックを書かずに手元の環境でも再現できる実装になったと思っています。
