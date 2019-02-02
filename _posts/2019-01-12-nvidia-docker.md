---
layout: post
date:   2019-1-12 15:00:00 -0900
title:  Ubuntu16.04でnvidia-dockerを動かす
description: Ubuntu16.04へのnvidia-dockerのインストール方法
categories: ubuntu
tags:
- ubuntu
- deep learning
---

### 構成
- CPU: Core i7-6850K
- GPU: ELSA GeForce GTX 1080 8GB GLADIAC *2台


### 初期設定
#### ネットワーク
```shell-session
$ sudo apt-get -y install resolvconf

$ sudo nmcli con mod eno1 ipv4.method manual
$ sudo nmcli con mod eno1 ipv4.address 192.168.1.23/24
$ sudo nmcli con mod eno1 ipv4.dns 192.168.1.1
$ sudo nmcli con mod eno1 ipv4.gateway 192.168.1.1

$ sudo nmcli con down eno1 && sudo nmcli con up eno1
```

#### アップデート
```shell-session
$ sudo apt-get update
$ sudo apt-get -y upgrade
$ sudo apt-get -y dist-upgrade
```

#### OpenSSHのインストール
```shell-session
$ sudo apt-get -y install openssh-server

$ sudo systemctl start sshd
$ sudo systemctl enable sshd
```

### CUDA
#### 確認
以下のコマンドで，何も出てこないことを確認する
```shell-session
$ sudo dpkg -l | grep nvidia
$ sudo dpkg -l | grep cuda
```

#### Nvidiaドライバのインストール
リポジトリ([Proprietary GPU Drivers : “Graphics Drivers” team](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa))  
を登録して，ドライバをインストールする  
途中で，secure bootを無効にするかを聞かれたが，無効にせずに続行した
```shell-session
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update

$ sudo apt-get -y install nvidia-387
```

再起動し，GPUが認識されているかを確認
```shell-session
$ sudo reboot

$ nvidia-smi
Sat Dec  9 10:19:09 2017       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 387.34                 Driver Version: 387.34                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 106...  Off  | 00000000:01:00.0  On |                  N/A |
|  0%   37C    P8     6W / 156W |     91MiB /  6071MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1130      G   /usr/lib/xorg/Xorg                            89MiB |
+-----------------------------------------------------------------------------+
```


### dockerのインストール
Ubuntu公式リポジトリの`docker.io`では`nvidia-docker`を実行できない
(`docker-engine`, `docker-ce`, `docker-ee`のいずれかのパッケージが依存関係として必要)
ので、[Get Docker CE for Ubuntu | Docker Documentation](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#set-up-the-repository)
の通りにインストールする

```shell-session
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update

$ sudo apt-get -y install docker-ce
```


### nvidia-dockerのインストール
[GitHub - NVIDIA/nvidia-docker: Build and run Docker containers leveraging NVIDIA GPUs](https://github.com/NVIDIA/nvidia-docker)の通りにインストールする

```shell-session
$ sudo apt-get -y install curl

$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
$ sudo apt-get update

$ sudo apt-get install -y nvidia-docker2
$ sudo pkill -SIGHUP dockerd
```

テスト(`nvidia-docker run --rm nvidia/cuda nvidia-smi`でもいける)

```shell-session
$ docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
Unable to find image 'nvidia/cuda:latest' locally
latest: Pulling from nvidia/cuda
660c48dd555d: Pull complete
4c7380416e78: Pull complete
421e436b5f80: Pull complete
e4ce6c3651b3: Pull complete
be588e74bd34: Pull complete
f597507b3c37: Pull complete
9c5d4127a23d: Pull complete
398bf259fcdc: Pull complete
4f4092762618: Pull complete
94130a21e154: Pull complete
Digest: sha256:954c82d2d060f38de13b3d7933b7e1549b25330cc6412008dc1253f3c148448d
Status: Downloaded newer image for nvidia/cuda:latest
Sat Dec  9 01:59:36 2017       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 387.34                 Driver Version: 387.34                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 106...  Off  | 00000000:01:00.0  On |                  N/A |
|  0%   37C    P8     6W / 156W |     91MiB /  6071MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```
