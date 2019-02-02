---
layout: post
date:   2019-2-2 00:00:00 +0900
title:  CentOS7に新しめのDocker環境を構築する
description: CentOS7へのDockerCE、docker-composeのインストール方法
categories: centos7
tags:
  - cenos7
  - docker
---


デフォルトのDockerが1.13.1で`FROM`前の`ARG`が使えなかったので、あたらしめのDockerを入れる。
```shell-session
$ sudo yum list docker
docker.x86_64    2:1.13.1-88.git07f3374.el7.centos    extras
```


### ホスト環境
```shell-session
$ uname -sr
Linux 3.10.0-957.1.3.el7.x86_64

$ cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```


### Dockerの削除
yumでDockerを入れていた場合は削除する
```shell-session
$ sudo yum -y remove docker docker-common
```


### Dockerのインストール
リポジトリを追加
```shell-session
$ sudo yum -y install yum-utils
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

`docker-ce`をインストール
```shell-session
$ sudo yum -y install docker-ce
```

おそらく`docker`グループが自動作成されているはずなので、ユーザを`docker`グループに追加する
```shell-session
$ cat /etc/group | grep docker
dockerroot:x:983:
docker:x:979:

$ sudo gpasswd -a <USER> docker
$ id
uid=1000(<USER>) gid=1000(<USER>) groups=1000(<USER>),10(wheel),979(docker)
```

デーモンを起動
```shell-session
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

ログインし直せば使える
```shell-session
$ docker --version
Docker version 18.09.1, build 4c52b90
```


### docker-composeのインストール
リポジトリがなさそうだったので公式（[Install Docker Compose | Docker Documentation](https://docs.docker.com/compose/install/)）の通りに下記コマンドでインストールする。

```shell-session
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ docker-compose --version
docker-compose version 1.23.2, build 1110ad01
```
