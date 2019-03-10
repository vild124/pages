---
layout: post
date:   2019-3-19 00:00:00 +0900
title:  Debian 9に最新のDockerを入れる
description: Debian9へdockerリポジトリを追加し最新のDocker環境を構築する
categories: debian
tags:
- linux
- debian
- docker
---


### dockerのインストール
[Get Docker CE for Debian | Docker Documentation](https://docs.docker.com/install/linux/docker-ce/debian/)のとおり、リポジトリを追加してインストールする。

```shell-session
$ sudo apt update
$ sudo apt -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
OK

$ sudo apt-key fingerprint 0EBFCD88
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

$ sudo apt update
$ sudo apt -y install docker-ce docker-ce-cli containerd.io
```


### docker-composeのインストール
リポジトリがなさそうなので直接インストールする。

```shell-session
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

### いろいろと設定
ユーザをdockerグループに追加し、dockerデーモンを自動起動するように設定する。
```shell-session
$ sudo gpasswd -a vild124 docker
Adding user vild124 to group docker

$ sudo systemctl enable docker
$ sudo reboot
```
