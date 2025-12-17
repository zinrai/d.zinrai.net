---
title: "Ansible の Template モジュールを使ったコンフィグをサーバーに適用する前にレンダリングして CI を可能にするツールを作ってみた"
date: 2025-12-11T21:41:46+09:00
---

[さくらインターネット Advent Calendar 2025](https://qiita.com/advent-calendar/2025/sakura) シリーズ 2 11日目の記事です。

[Ansible](https://github.com/ansible/ansible) の [Template モジュール](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/template_module.html) を適用前にレンダリングして CI で検証可能にするツールを作ってみました。

[zinrai/ansible-template-render](https://github.com/zinrai/ansible-template-render)

## 使い方

* Ansible のコマンドを利用しているため [Ansible](https://github.com/ansible/ansible) をインストールします
* [zinrai/ansible-template-render](https://github.com/zinrai/ansible-template-render) から ansible-template-render のバイナリをダウンロードしてパスの通った場所に配置します

リポジトリ内に example を用意していて、どんな感じになるかを試せるようになっています。

```
$ cd example
$ cat render-config.yaml
playbooks:
  - name: site
    inventory: inventory
options:
  ansible_args: "--diff"
$ ls
inventory  render-config.yaml  roles  site.yml
```

```
$ ansible-template-render -config render-config.yaml
time=2025-12-17T22:17:18.808+09:00 level=INFO msg="Processing playbook" name=site
time=2025-12-17T22:17:18.808+09:00 level=INFO msg="Found playbook" path=site.yml
time=2025-12-17T22:17:18.808+09:00 level=INFO msg="Found inventory" path=inventory
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="Converted inventory for local execution" path=tmp-site/inventory.yaml
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="Found direct roles" roles=[webserver]
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="All roles (including dependencies)" roles=[webserver]
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="Copying role" name=webserver
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="Processing role tasks" name=webserver
time=2025-12-17T22:17:19.142+09:00 level=INFO msg="Modified template task" file=roles/webserver/tasks/main.yml
time=2025-12-17T22:17:19.143+09:00 level=INFO msg="Executing Ansible command" command="ansible-playbook site.yml --tags render_config -i inventory.yaml --diff"

PLAY [Configure webservers] ****************************************************************************

TASK [Gathering Facts] *********************************************************************************
[WARNING]: Host 'web01' is using the discovered Python interpreter at '/usr/bin/python3.13', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.19/reference_appendices/interpreter_discovery.html for more information.
ok: [web01]
[WARNING]: Host 'web02' is using the discovered Python interpreter at '/usr/bin/python3.13', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.19/reference_appendices/interpreter_discovery.html for more information.
ok: [web02]

TASK [webserver : Ensure directory exists for output/etc/nginx/nginx.conf] *****************************
--- before
+++ after
@@ -1,5 +1,5 @@
 {
-    "mode": "0775",
+    "mode": "0755",
     "path": "output/etc/nginx",
-    "state": "absent"
+    "state": "directory"
 }

changed: [web01 -> localhost]

TASK [webserver : Configure nginx] *********************************************************************
--- before
+++ after: /home/nanashi/src/github-src/ansible-template-render/example/tmp-site/ansible-tmp/ansible-local-13592236uviusap/tmpxovbnj38/nginx.conf.j2
@@ -0,0 +1,32 @@
+user nginx;
+worker_processes auto;
+error_log /var/log/nginx/error.log;
+pid /run/nginx.pid;
+
+events {
+    worker_connections 1024;
+}
+
+http {
+    include /etc/nginx/mime.types;
+    default_type application/octet-stream;
+
+    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
+                    '$status $body_bytes_sent "$http_referer" '
+                    '"$http_user_agent" "$http_x_forwarded_for"';
+
+    access_log /var/log/nginx/access.log main;
+
+    sendfile on;
+    keepalive_timeout 65;
+
+    server {
+        listen 80;
+        server_name sakura;
+
+        location / {
+            root /usr/share/nginx/html;
+            index index.html;
+        }
+    }
+}

changed: [web01 -> localhost]

PLAY RECAP *********************************************************************************************
web01                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
web02                      : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

time=2025-12-17T22:17:21.409+09:00 level=INFO msg="Templates successfully rendered" output=tmp-site/output
time=2025-12-17T22:17:21.409+09:00 level=INFO msg="Successfully generated template files"
$
```

```
$ find tmp-site/
tmp-site/
tmp-site/ansible.cfg
tmp-site/ansible-tmp
tmp-site/inventory.yaml
tmp-site/roles
tmp-site/roles/webserver
tmp-site/roles/webserver/templates
tmp-site/roles/webserver/templates/etc
tmp-site/roles/webserver/templates/etc/nginx
tmp-site/roles/webserver/templates/etc/nginx/nginx.conf.j2
tmp-site/roles/webserver/tasks
tmp-site/roles/webserver/tasks/main.yml
tmp-site/roles/webserver/defaults
tmp-site/roles/webserver/defaults/main.yml
tmp-site/output
tmp-site/output/etc
tmp-site/output/etc/nginx
tmp-site/output/etc/nginx/nginx.conf
tmp-site/site.yml
$
```

## 仕組み

Ansible は YAML なので YAML をパースして Template モジュールの宣言にマッチしたら [delegate_to](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_delegation.html#delegating-tasks) と [run_once](https://docs.ansible.com/projects/ansible/latest/reference_appendices/playbooks_keywords.html) と render_config という名前の tag を挿入した playbook を生成し、生成した playbook にて render_config の tag が付いた task のみを実行する ansible-playbook コマンドを実行することで Template モジュールを使ったコンフィグがローカルに展開されるという仕組みになっています。

[Ansible Best Practice](https://docs.ansible.com/projects/ansible/2.9/user_guide/playbooks_best_practices.html) のディレクトリ構成であることを期待しています。

ansible-inventory コマンドを内部で仕様しており、 INI 形式の inventory も YAML 形式の inventory も Dynamic Inventory も YAML の static inventory として出力され ansble-plahybook コマンドが実行されるようにているため static inventory と dynamic inventory どちらにも対応しています。

[community.general.consul_kv lookup](https://docs.ansible.com/projects/ansible/latest/collections/community/general/consul_kv_lookup.html) のように外部のストレージから値を取得してくるようなコードが playbook 内に書かれている場合は、外部ストレージへの到達性が無いと ansible-playbook コマンドの実行に失敗します。

## なんで作ったのか

Ansible の Template モジュールは ansible-playbook コマンドを実行しターゲットにコンフィグが適用されるタイミングで Template で記述されたコンフィグの内容が確定するため、サーバーに適用する前に生成されたコンフィグを確認するということができず CI しにくいなとずっと思っていました。

Ansible のドキュメントを眺めてみても Template モジュールの内容をローカルに展開するようなツールは自分が調べた限りでは存在せず delegate_to ディレクティブしかなかったため、 ansible playbook の YAML を書き換えて ansible-playbook コマンドを実行するツールを自作するしかないと考え作りました。

Ansible で構成管理していて Template モジュールを利用しているコンフィグを GitHub Actions 上で ansible-template-render を使い展開し CI できるようになりました。
