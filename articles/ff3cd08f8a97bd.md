---
title: "Caddy：ローカルのファイルを https://localhost/xxx でアクセスできるようにする"
emoji: "🙆"
type: "tech"
topics: ["web", "caddy"]
published: true
---

# はじめに
ローカルにあるファイルを、https://localhost/test.jpg のようなURLでアクセスできるようにしたいです。
できるだけ簡単に環境構築するため、 [Caddy](https://caddyserver.com/) というWebサーバを利用しました。

# Caddyとは
Go言語で書かれたオープンソースのWebサーバです。
https://caddyserver.com/

CaddyはデフォルトでHTTPSを有効にします。この機能が便利だと思い、Caddyを使いました。

# Caddyfile
Caddyの設定は[Caddyfile](https://caddyserver.com/docs/caddyfile)というファイルに記述します。

```plain:Caddyfile
# ホスト名がlocalhost
localhost
# `/tmp/www` 配下をファイルサーバとして公開する
root * /tmp/www
file_server {
	# ファイル一覧を表示する
	browse
}
```

`/tmp/www`の中身は以下の通りです。

```
$ ls /tmp/www
hello.txt
$ cat /tmp/www/hello.txt 
hello world
```

# caddyの起動
[caddy run](https://caddyserver.com/docs/command-line#caddy-run)コマンドで、webサーバを起動します。

```
$ sudo caddy run -config Caddyfile 
2021/09/20 05:21:29.207	INFO	using provided configuration	{"config_file": "Caddyfile", "config_adapter": ""}
2021/09/20 05:21:29.209	INFO	admin	admin endpoint started	{"address": "tcp/localhost:2019", "enforce_origin": false, "origins": ["localhost:2019", "[::1]:2019", "127.0.0.1:2019"]}
2021/09/20 05:21:29.209	INFO	http	server is listening only on the HTTPS port but has no TLS connection policies; adding one to enable TLS	{"server_name": "srv0", "https_port": 443}
2021/09/20 05:21:29.210	INFO	http	enabling automatic HTTP->HTTPS redirects	{"server_name": "srv0"}
2021/09/20 05:21:29.209	INFO	tls.cache.maintenance	started background certificate maintenance	{"cache": "0xc0001dcc40"}
2021/09/20 05:21:29.211	INFO	tls	cleaning storage unit	{"description": "FileStorage:/home/vagrant/.local/share/caddy"}
2021/09/20 05:21:29.211	INFO	tls	finished cleaning storage units
2021/09/20 05:21:29.229	INFO	pki.ca.local	root certificate is already trusted by system	{"path": "storage:pki/authorities/local/root.crt"}
2021/09/20 05:21:29.229	INFO	http	enabling automatic TLS certificate management	{"domains": ["localhost"]}
2021/09/20 05:21:29.229	WARN	tls	stapling OCSP	{"error": "no OCSP stapling for [localhost]: no OCSP server specified in certificate"}
2021/09/20 05:21:29.229	INFO	autosaved config (load with --resume flag)	{"file": "/home/vagrant/.config/caddy/autosave.json"}
2021/09/20 05:21:29.229	INFO	serving initial configuration

```

## バックグラウンドで起動する場合
`caddy run`コマンドはフォアグランドでプロセスを起動します。バックグランドでプロセスを起動する場合は、[caddy start](https://caddyserver.com/docs/command-line#caddy-start)コマンドを実行します。


# 動作確認

https://localhost/hello.txt にアクセスして、ファイルの中身を取得できます。

```
$ curl https://localhost/hello.txt
hello world
```

また、https://localhost にアクセスすると、`/tmp/www`配下のファイルリストが表示されます。

![](https://storage.googleapis.com/zenn-user-upload/87f2f1ec85b148a864356874.png)

# 補足

## レスポンスヘッダを付与する
[eaderディレクティブ](ttps://caddyserver.com/docs/caddyfile/directives/header)で、レスポンスヘッダを付与できます。

```plain:Caddyfile
header {
    Access-Control-Allow-Origin: https://example.com
}
```

## Caddyの設定の確認

http://localhost:2019/config/ [^1] にアクセスすると、現在のCaddyの設定を確認できます。Caddyfileに書いた通りの動きでない場合、まずはこの方法で設定を確認するのが良いと思います。

[^1]: URLの末尾にスラッシュが必要です。http://localhost:2019/config だと、アクセスできません。

https://caddyserver.com/docs/api#get-configpath

```
$ curl http://localhost:2019/config/ | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   277  100   277    0     0  69250      0 --:--:-- --:--:-- --:--:-- 69250
{
  "apps": {
    "http": {
      "servers": {
        "srv0": {
          "listen": [
            ":443"
          ],
          "routes": [
            {
              "handle": [
                {
                  "handler": "subroute",
                  "routes": [
                    {
                      "handle": [
                        {
                          "handler": "vars",
                          "root": "/tmp/www"
                        },
                        {
                          "browse": {},
                          "handler": "file_server",
                          "hide": [
                            "./Caddyfile"
                          ]
                        }
                      ]
                    }
                  ]
                }
              ],
              "match": [
                {
                  "host": [
                    "localhost"
                  ]
                }
              ],
              "terminal": true
            }
          ]
        }
      }
    }
  }
}
```

## https://127.0.0.1/ にはアクセスできない
ほとんどの環境では `localhost`は`127.0.0.1`と同じなので、https://127.0.0.1/ でもアクセスできそうですが、Caddyの場合はアクセスできません。リクエストヘッダの`Host`を確認しているためです。
詳細は https://zenn.dev/yuji38kwmt/articles/5d3e729dc9a5c1 を参照してください。




