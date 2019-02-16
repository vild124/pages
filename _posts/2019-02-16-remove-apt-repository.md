---
layout: post
date:   2019-2-16 00:00:00 -0900
title:  Ubuntuで不要になったリポジトリを削除する
description: aptで不要なリポジトリを削除する方法
categories: ubuntu
tags:
- linux
- ubuntu
---


`sudo apt update`するといろいろとエラーが出るようになったので、使っていないnvidia-dockerのリポジトリを削除する。  
ついでに、Nvidiaのグラフィックドライバのリポジトリも削除する


### nvidia-dockerの削除
まずnvidia-docker2自体を削除する
```shell-session
$ sudo apt remove --purge nvidia-docker2
$ sudo apt -y autoremove
```

リポジトリリストを削除する
```shell-session
$ cd /etc/apt/sources.list.d
$ ls 
graphics-drivers-ubuntu-ppa-xenial.list       nvidia-docker.list
graphics-drivers-ubuntu-ppa-xenial.list.save  nvidia-docker.list.save

$ sudo rm nvidia-docker.list nvidia-docker.list.save
```

GPGキーを削除する
```shell-session
$ sudo apt-key list
/etc/apt/trusted.gpg
--------------------
...
pub   4096R/F796ECB0 2017-09-28
uid                  NVIDIA CORPORATION (Open Source Projects) <cudatools@nvidia.com>
...

$ sudo apt-key del F796ECB0
OK
```
これで`apt-key list`でNvidiaのキーが表示されなくなる

updateしてみてもエラーが出なくなるはず
```shell-session
$ sudo apt update
Hit:1 http://jp.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://jp.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]                            
Get:3 http://jp.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]                                                
Get:4 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]                                                   
Hit:5 http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu xenial InRelease       
Get:6 https://download.docker.com/linux/ubuntu xenial InRelease [66.2 kB]         
Get:7 http://security.ubuntu.com/ubuntu xenial-security/main amd64 DEP-11 Metadata [67.9 kB]
Get:8 http://security.ubuntu.com/ubuntu xenial-security/main DEP-11 64x64 Icons [67.1 kB]
Get:9 http://security.ubuntu.com/ubuntu xenial-security/universe amd64 DEP-11 Metadata [116 kB]
Get:10 http://security.ubuntu.com/ubuntu xenial-security/universe DEP-11 64x64 Icons [173 kB]
Get:11 http://security.ubuntu.com/ubuntu xenial-security/multiverse amd64 DEP-11 Metadata [2,464 B]
Fetched 818 kB in 2s (312 kB/s)     
Reading package lists... Done
Building dependency tree       
Reading state information... Done
```


### graphics-driversの削除
追加時にコマンド
```shell-session
$ sudo add-apt-repository ppa:graphics-drivers/ppa
```
でppaを使って追加したので
```shell-session
$ sudo add-apt-repository --remove ppa:graphics-drivers/ppa
```
だけで削除できる。

updateしてもちゃんと消えている
```shell-session
$ sudo apt update
Hit:1 http://jp.archive.ubuntu.com/ubuntu xenial InRelease
Get:2 http://jp.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]                           
Get:3 http://jp.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]                                   
Hit:4 https://download.docker.com/linux/ubuntu xenial InRelease                                                  
Hit:5 http://security.ubuntu.com/ubuntu xenial-security InRelease                                                
Fetched 216 kB in 0s (274 kB/s)
Reading package lists... Done
Building dependency tree       
Reading state information... Done
```
