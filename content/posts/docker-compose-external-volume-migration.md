---
title: "Docker Compose の external volumes をホスト間で移行するツールを作ってみた"
date: 2025-12-20T13:09:39+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 20日目の記事です。

Docker Compose の external volumes をホスト間で移行するツールを作ってみました。

[zinrai/compose-volume-migrate](https://github.com/zinrai/compose-volume-migrate)

## 使い方

[zinrai/redmine-docker-compose](https://github.com/zinrai/redmine-docker-compose) を例に使い方を説明します。

移行元から external volume を export する。

```
$ sudo docker compose ps
NAME               IMAGE                  COMMAND                  SERVICE    CREATED              STATUS                        PORTS
redmine-app        redmine:6.1.0-trixie   "/docker-entrypoint.…"   redmine    About a minute ago   Up 51 seconds                 127.0.0.1:3000->3000/tcp
redmine-postgres   postgres:17.6-trixie   "docker-entrypoint.s…"   postgres   About a minute ago   Up About a minute (healthy)   5432/tcp
$ sudo docker compose stop
[+] Stopping 2/2
 ✔ Container redmine-app       Stopped                                                             0.2s
 ✔ Container redmine-postgres  Stopped                                                             0.2s
$ sudo compose-volume-migrate export
Exporting postgres_data
Exporting redmine_files
Exporting redmine_plugins
$ ls -1
LICENSE
README.md
docker-compose.yaml
postgres_data.tar.gz
redmine_files.tar.gz
redmine_plugins.tar.gz
scripts
$
```

移行元から export した external volume を移行先に import する

```
$ sudo docker volume ls
DRIVER    VOLUME NAME
$ ls -1
LICENSE
README.md
docker-compose.yaml
postgres_data.tar.gz
redmine_files.tar.gz
redmine_plugins.tar.gz
scripts
$ sudo compose-volume-migrate import
Importing postgres_data
Importing redmine_files
Importing redmine_plugins
$ sudo docker volume ls
DRIVER    VOLUME NAME
local     postgres_data
local     redmine_files
local     redmine_plugins
$ sudo docker compose up -d
[+] Running 3/3
 ✔ Network redmine-docker-compose_redmine_network  Created                                         0.1s
 ✔ Container redmine-postgres                      Healthy                                         0.1s
 ✔ Container redmine-app                           Started                                         0.0s
$ sudo docker compose ps
NAME               IMAGE                  COMMAND                  SERVICE    CREATED          STATUS                    PORTS
redmine-app        redmine:6.1.0-trixie   "/docker-entrypoint.…"   redmine    26 seconds ago   Up 14 seconds             127.0.0.1:3000->3000/tcp
redmine-postgres   postgres:17.6-trixie   "docker-entrypoint.s…"   postgres   26 seconds ago   Up 25 seconds (healthy)   5432/tcp
$
```

## なんで作ったのか

個人的に運用しているサーバーの中に Docker Compose で動かしているアプリケーションがあり docker compose down -v したときに消えてほしくない volume を external volume として管理していて、サーバー移行時に external volume を移行する対象が複数ありチマチマとした移行のオペレーションが面倒だなと思い作りました。

docker-compose.yaml をパースして external volume のみを抽出して中身をアーカイブしたり、移行先に external volume を作成しています。移行元にも移行先にも Docker があるので busybox の Docker コンテナイメージを使い external volume の中身をアーカイブしています。
