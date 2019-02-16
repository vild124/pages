---
layout: post
date:   2019-2-16 00:00:00 -0900
title:  OpenVPNの"No server certificate verification method has been enabled"を解決する
description: OpenVPNの警告"No server certificate verification method has been enabled"が出ないように設定を変更する
categories: openvpn
tags:
- linux
- docker
- openvpn
---

{{ site.baseurl }}{% post_url 2019/02/2019-02-15-docker-openvpn %}
で構築したOpenVPNサーバに接続する時に接続はできるが下の警告が出るので、出ないように設定したい。

__警告内容__
```
WARNING: No server certificate verification method has been enabled.
See http://openvpn.net/howto.html#mitm for more info.
```


### 解決方法
メッセージに出ていたURLのページ（[2x HOW TO | OpenVPN](https://openvpn.net/community-resources/how-to/#mitm)）を見てみる

- `remote-cert-tls server`
    - OpenVPN 2.0以下の場合はこの設定をクライアントファイルに追加する?
- `ns-cert-type server`
    - それ以外のバージョン?ではこの設定をクライアントファイルに追加する?

特にサーバ側の設定は必要なさそう。


### ns-cert-type server
OpenVPNサーバのバージョンが__2.4.6__なのでこっちを試してみる。

クライアントの*.ovpnファイルの適当な位置に`ns-cert-type server`を追加
```ovpn
client
dev tun
proto udp
ns-cert-type server
...省略
```

接続してみると
```
WARNING: -ns-cert-type is DEPRECATED. Use --remote-cert-tls instead.
```
が表示されて接続できなくなった。



### remote-cert-tls server
*.ovpnファイルの
```
ns-cert-type server
```
を
```
remote-cert-tls server
```
に書き換えて接続すると警告がなくなって接続できるようになった。
