---
layout: post
date:   2019-1-21 00:00:00 +0900
title:  KerasがGPUを認識できなくなった
description: Ubuntu18.04のnvidia-docker上で動かしているKerasがCPUでしか動かなくなった
categories: ubuntu
tags:
  - ubuntu
  - docker
---


2018年12月時点ではGPUで動いていたコードが2019年1月になるとGPUを認識できずにCPUで動くようになっていた。  
Ubuntu自体のアップデート、Pythonパッケージのアップデートをしていた気がするのでそれが原因か?  
記事執筆時点（2019年1月21日）での各バージョンは下記の通り。

- Ubuntu 18.04.1 LTS
    - Linux 4.15.0-43-generic
    - nvidia-396/now 396.54-0ubuntu0~gpu16.04.1 amd64
- Docker
    - docker-ce/bionic,now 18.06.1~ce~3-0~ubuntu amd64
    - nvidia-docker2/now 2.0.3+docker18.06.1-1 all
- pip
    - Keras==2.0.4
    - tensorflow-gpu==1.12.0


学習を開始すると
```shell-session
failed call to cuInit: CUDA_ERROR_NO_DEVICE: no CUDA-capable device is detected
```
が出て学習自体は動くが、GPUを使っていないため遅い。



### GPUドライバの再インストール
__最終的にはGPUドライバの再インストールはおそらく不要?__

```
nvidia-396/now 396.54-0ubuntu0~gpu16.04.1 amd64
```
の`gpu16.04.1`部分がUbuntuのバージョンを16.04から`dist-upgrade`したときの残骸?のような感じで怪しいのでGPUドライバを再インストールしてみる。

削除後、一応再起動。
```shell-session
$ sudo apt remove nvidia-396
$ sudo apt autoremove

$ sudo reboot
```

アップデート後、最新版を確認してみる
```shell-session
$ sudo apt update
$ sudo apt search nvidia
nvidia-396/now 396.54-0ubuntu0~gpu16.04.1 amd64 [設定が残存]
  (none)
```
` [設定が残存]`になっているので
```shell-session
$ sudo apt purge nvidia-396
$ sudo reboot
```
完全に消えたのでppaを追加してインストールする。  
nvidia-396はリポジトリにはなかったので、代わりにnvidia-390をインストールする。
```shell-session
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt update
$ sudo apt install nvidia-390
```


---
### CUDA_VISIBLE_DEVICESが原因
コードのはじめの方でGPUが一枚しかないのに
```python
os.environ["CUDA_VISIBLE_DEVICES"] = "1"
```
としていたことが原因。

```python
os.environ["CUDA_VISIBLE_DEVICES"] = "0"
```
やCUDA_VISIBLE_DEVICESを設定しなければ問題なくGPUで動いた。
