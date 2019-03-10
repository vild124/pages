---
layout: post
date:   2019-3-9 00:00:00 +0900
title:  Debian Stretchインストール後の設定
description: Debian StretchでIP固定する方法など
categories: debian
tags:
- linux
- debian
---


### リポジトリの設定
リポジトリのリストからcdromを削除する。  
sudoが使えないのでsuしてから作業する。
```shell-session
$ su

# sed -i -e 's/^deb cdrom/# deb cdrom/' /etc/apt/sources.list
```

一応アップデートしておく
```shell-session
# apt update
# apt -y upgrade
```


### sudoの有効化
sudoのコマンドのインストール
```shell-session
$ su
# apt -y install sudo 
```

ユーザをsudoグループに追加し、sudoできるようにする。
```shell-session
# gpasswd -a ユーザ名 sudo
```
ログインし直すとsudoできるようになる。



### IPの固定
`static`に変更し、IPアドレス等を追加。
```
$ sudo vim /etc/network/interfaces
#iface eth0 inet dhcp
iface eth0 inet static
address         192.168.1.5
network         192.168.1.0
netmask         255.255.255.0
broadcast       192.168.1.255
gateway         192.168.1.1
dns-nameservers 192.168.1.1

$ sudo ifdown eth0 && sudo ifup eth0
```
