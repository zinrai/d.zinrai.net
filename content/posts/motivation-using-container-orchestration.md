---
title: "コンテナオーケストレーションを使うモチベーションを自分なりに考えてみた"
date: 2021-12-06T10:26:12+09:00
---

[さくらインターネット Advent Calendar 2021](https://qiita.com/advent-calendar/2021/sakura) 6日目の記事になります。

[Docker でのアプリケーション実行環境を提供した状況を想像してみる](/posts/imagine-situation-using-docker/) にて、 Docker を導入しただけで、万事解決とはならなそうであるということを書きました。

今回は、 Docker の導入だけでは解決できないことに対して、どのようなことができればよいのかということを考え、それらを解決する具体的な手段について書いていきます。

# Docker だけでは解決できていないこと

下記のようなことができれば、アプリケーション開発者はサーバーのことを気にせず、
サーバー管理者はアプリケーションのことを気にしない仕組みになるのではないかと考えました。

「 Docker を実行できる環境があれば、どこでもアプリケーションを実行できるが、サーバーで Docker を実行できるだけでは、サーバーに付けた名前や IP アドレスで、アプリケーションに接続するという構図から抜け出せない」

Docker を実行できるサーバーを沢山用意して、その中のどれかでアプリケーションが実行されるということができれば、
アプリケーションとサーバーが密に結合するという状態を解消できそうです。

「サーバー管理者の裁量でサーバーをコントロールできる部分が少なそう」

Docker を実行できるサーバーを沢山用意して、
その中のどれかでアプリケーションが実行されるということができるかつ、
メンテナンスしたいサーバーでは Docker コンテナを実行できないようにすれば、
アプリケーション開発者は、サーバーの停止を意識することがなくアプリケーションを実行でき、
サーバー管理者は、サーバーを任意のタイミングで停止できるようになりそうです。

「 Docker で動くアプリケーションをデプロイする統一されたインタフェースが無い」

定義ファイルを書いて実行すれば Docker で動くアプリケーションがデプロイされる仕組みがあれば、
構成管理もできてよさそうです。
アプリケーション開発者は、定義ファイルを書いて実行することでアプリケーションを実行し、
サーバー管理者は、 Ansible でサーバーを構成管理することで、
アプリケーションとサーバー管理の手段を分離できそうです。

# Nomad を使えば実現できる

Docker だけでは解決できていないことに対して、どのようなことができればよいかを考えてみました。
これらは Docker に Nomad というソフトウェアを組み合わせることで、実現できます。

「Nomad とは」というお話は、別の記事や公式ドキュメントに任せ、
Nomad と Docker を組み合わせることで何ができるのかについて、書いていきます。

https://www.nomadproject.io

## アーキテクチャ

https://learn.hashicorp.com/tutorials/nomad/production-reference-architecture-vm-with-consul#reference-diagram

![Reference diagram](../../imgs/motivation-using-container-orchestration/nomad_reference_diagram.png)

緑色の Nomad と書かれているところだけを説明していきます。
( 赤色の Consul と書かれているところは、今回は気にしないでください。 )

Nomad は、 Nomad Server, Nomad Client の構成で動作します。
Docker を利用する今回のケースでは、 Nomad Client で、 Docker で動くアプリケーションが起動し、
Nomad Server は、どの Nomad Client に Docker で動くアプリケーションを起動するのかを管理します。

## Job

Nomad では、どのようなアプリケーションを実行するかを定義ファイルとして書き、
Nomad に登録することで、 Nomad Client でアプリケーションが実行されます。
Nomad では、定義ファイルのことを Job と呼んでいます。

https://www.nomadproject.io/docs/internals/architecture#job

下記は、 http-echo というアプリケーションを Docker で実行する Job になります。

https://learn.hashicorp.com/tutorials/nomad/jobs-submit

```
job "docs" {
  datacenters = ["dc1"]

  group "example" {
    network {
      port "http" {
        static = "5678"
      }
    }
    task "server" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        ports = ["http"]
        args = [
          "-listen",
          ":5678",
          "-text",
          "hello world",
        ]
      }
    }
  }
}
```

# Nomad を使うことで何が起きたか

Nomad によって、 Docker を実行できるサーバーを Nomad Client として沢山並べ、
どれかの Nomad Client で Docker で動くアプリケーションが実行できるようになり、
アプリケーションとサーバーが密に結合することは無くなります。

Nomad の Drain という機能を利用すると、
特定の Nomad Client を Docker で動くアプリケーションを実行する対象から除外できます。
アプリケーション開発者は、サーバーが停止することを意識せずにアプリケーションを実行でき、
サーバー管理者は、任意のタイミングでサーバーをメンテナンスすることができます。

https://learn.hashicorp.com/tutorials/nomad/node-drain

アプリケーションをデプロイするインタフェースが Job という形で統一され、構成管理も可能になりました。
アプリケーション開発者は、 Job を書いてアプリケーションを実行し、
サーバー管理者は、 Ansible でサーバーを構成管理することで、
アプリケーションとサーバー管理の手段を分離できます。

Docker だけでは解決できなかったことが、 Nomad を組み合わせることにより解決しました。

そして「さくらの専用サーバ PHY」のアプリケーション実行基盤は、 Nomad と Docker によって支えられています。

# Nomad を使えば万事解決か

Nomad を使うことで、 Docker を実行できるサーバーを沢山並べてアプリケーションを実行できるようになり、
アプリケーションとサーバーが密に結合しなくなったことはわかったけれども、
どの Nomad Client で起動しているのかわらない状態で、アプリケーションに
どのようにアクセスすればよいのかという疑問が出てきているのではないかと思うので、その疑問には次回、答えていきます。
