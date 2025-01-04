---
title: "Hugo"
date: 2019-01-19T21:47:42+09:00
tags: ['hugo', 'debian']
---

debian-devel-changes のメーリングリストを眺めていたら、 HUGO という Site Generator を見付けた。
数年前にブログ環境を消失してから、ブログを書く気力が無くなってしまっていたが、HUGO を使い Github Pages にブログを書くことを再開しようと思う。

debian-devel-changes

https://lists.debian.org/debian-devel-changes/

HUGO

https://gohugo.io/

HUGO の Debian パッケージ

https://packages.debian.org/sid/hugo

## 実行環境
```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux buster/sid
Release:        unstable
Codename:       sid
```

```
$ uname -a
Linux koyomi 4.19.0-4-amd64 #1 SMP Debian 4.19.28-2 (2019-03-15) x86_64 GNU/Linux
```

## インストール
```
# apt-get install hugo
```

## バージョン
```
$ hugo version
Hugo Static Site Generator v0.54.0/extended linux/amd64 BuildDate: 2019-02-01T18:13:18Z
```

ブログのような形式で情報が並ぶと目的の情報に辿り着くまでがしんどかったので、
タグも日時も表示されず、単一のページにタイトルだけがずらっと並び、
そこを Ctrl + f で検索するような、とにかくシンプルなテーマを探し、下記を見付けた。

https://themes.gohugo.io/elephants/

クイックスタートに従いサイトを作っていく。

https://gohugo.io/getting-started/quick-start/


## サイトの作成
```
$ hugo new site d.zinrai.net
$ cd d.zinrai.net
$ git init
```

## テーマの取得
```
$ git submodule add https://gitlab.com/meibenny/elephants themes/elephants
```

## テーマの設定
```
$ cat << EOF >> config.toml
baseURL = "/"
relativeurls = true
languageCode = "ja"
title = "command not found:"
theme = "elephants"
EOF
```

## 記事作成
```
$ hugo new posts/hugo.md
```

## 内容確認
```
$ hugo server -D
```

## 記事生成
```
$ hugo

                   | EN
+------------------+----+
  Pages            |  9
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  1
  Processed images |  0
  Aliases          |  0
  Sitemaps         |  1
  Cleaned          |  0

Total in 8 ms
```

## Github への push

```
$ git add .
$ git commit .
$ git remote add origin git@github.com/zinrai/d.zinrai.net.git
$ git push -u origin master
```
