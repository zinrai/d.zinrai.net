---
title: "Git リポジトリから特定のファイルを同期するツールを作ってみた"
date: 2025-12-13T09:23:04+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 13日目の記事です。

Git リポジトリから特定のファイルを同期するツールを作ってみました。

[zinrai/git-file-sync](https://github.com/zinrai/git-file-sync)

## 使い方

* 内部で git コマンドを利用しているため git のインストールが必要です
* [zinrai/git-file-sync](https://github.com/zinrai/git-file-sync) のバイナリをダウンロードしパスの通った場所に配置します

```
$ cat << EOF > config.yaml
git:
  repoUrl: "git@github.com:zinrai/git-file-sync.git"
  commitHash: "48a8bc8f8f00340c4902134b0915628ed300b340"

files:
  - source: "README.md"
    destination: "/tmp/README.md"

sshPrivateKeyPath: "/home/nanashi/.ssh/id_ecdsa"
EOF
$
```

```
$ ./git-file-sync -config config.yaml
time=2025-12-18T09:45:56.377+09:00 level=INFO msg="starting sync" repo=git@github.com:zinrai/git-file-sync.git commit=48a8bc8f8f00340c4902134b0915628ed300b340 files=1
time=2025-12-18T09:46:10.603+09:00 level=INFO msg=copied source=README.md destination=/tmp/README.md
time=2025-12-18T09:46:10.603+09:00 level=INFO msg="sync completed successfully"
$
```

```
$ head /tmp/README.md
# git-file-sync

A tool to sync specific files from a Git repository to local filesystem.

## Features

- Fetch specific files from a Git repository using commit hash
- Support for SSH authentication with deploy keys
- One-shot or daemon mode operation
- YAML configuration file
$
```

## なんで作ったのか

[HashiCorp Nomad](https://github.com/hashicorp/nomad) にて [Download using git](https://developer.hashicorp.com/nomad/docs/job-specification/artifact#download-using-git) という Git リポジトリを取得できる機能が提供されていて、リポジトリの取得はできるがリポジトリ内の特定のファイルやディレクトリを取得するといった機能は無さそうで、 Git リポジトリ内の特定のファイルやディレクトリを取得するツールが OSS で公開されていないか調べてみたところ、自分が調べた範囲では見当らなかったので作ってみました。

partial clone と sparse-checkout を組み合わせて、必要なファイルだけを取得しているため、リポジトリ全体を clone するよりもダウンロード量を抑えられる実装にしています。
