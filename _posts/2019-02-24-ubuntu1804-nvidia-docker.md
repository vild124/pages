---
layout: post
date:   2019-02-24 00:00:00 +0900
title:  Ubuntu18.04でnvidia-dockerを動かす
description: Ubuntu18.04へのnvidia-dockerのインストール方法
categories: ubuntu
tags:
- ubuntu
- deep learning
---


[Ubuntu16.04でnvidia-dockerを動かす]({{ site.baseurl }}{% post_url 2019-01-12-ubuntu1604-nvidia-docker %})のUbuntu18.04版。

### 構成
- CPU: AMD Ryzen Threadripper 1950X
- GPU: MSI GeForce GTX 1080ti 8GB
- OS: Ubuntu Server 18.04


### 初期設定
インストール時に  
- IPの固定
- SSHサーバの自動起動
は設定済みなのでアップデートのみ行う。

```shell-session
$ sudo apt update
$ sudo apt -y upgrade
```


---
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
$ sudo apt update

$ sudo apt search nvidia | egrep '^nvidia-[0-9]+/'
nvidia-304/bionic 304.137-0ubuntu2 amd64
nvidia-331/bionic-updates 340.107-0ubuntu0.18.04.2 amd64
nvidia-340/bionic-updates 340.107-0ubuntu0.18.04.2 amd64
nvidia-346/bionic 352.63-0ubuntu3 amd64
nvidia-352/bionic 361.45.11-0ubuntu4 amd64
nvidia-361/bionic 367.57-0ubuntu5 amd64
nvidia-367/bionic 375.82-0ubuntu3 amd64
nvidia-375/bionic 384.111-0ubuntu1 amd64
nvidia-384/bionic 390.87-0ubuntu0~gpu18.04.2 amd64
nvidia-387/bionic 390.87-0ubuntu0~gpu18.04.2 amd64
nvidia-390/bionic 390.87-0ubuntu0~gpu18.04.2 amd64

$ sudo apt -y install nvidia-390
```

再起動し，GPUが認識されているかを確認
```shell-session
$ sudo reboot

$ nvidia-smi
Fri Feb 22 04:36:58 2019
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.87                 Driver Version: 390.87                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:41:00.0 Off |                  N/A |
|  0%   38C    P5    27W / 250W |      0MiB / 11156MiB |      3%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```


### dockerのインストール
`docker-engine`, `docker-ce`, `docker-ee`のいずれかのパッケージが依存関係として必要なので、Ubuntu公式リポジトリの`docker.io`では`nvidia-docker`を実行できない。  
[Get Docker CE for Ubuntu | Docker Documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
の通りにインストールする

```shell-session
$ sudo apt -y remove docker docker-engine docker.io containerd runc

$ sudo apt -y install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt update

$ sudo apt -y install docker-ce docker-ce-cli containerd.io
```

```shell-session
$ sudo gpasswd -a <ユーザ名> docker

$ sudo systemctl start docker
$ sudo systemctl enable docker

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

```


### nvidia-dockerのインストール
[GitHub - NVIDIA/nvidia-docker: Build and run Docker containers leveraging NVIDIA GPUs](https://github.com/NVIDIA/nvidia-docker)の通りにインストールする

```shell-session
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu18.04/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
$ sudo apt update

$ sudo apt -y install nvidia-docker2
$ sudo pkill -SIGHUP dockerd
```

テスト
```shell-session
$ docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
Fri Feb 22 04:42:24 2019
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 390.87                 Driver Version: 390.87                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  Off  | 00000000:41:00.0 Off |                  N/A |
|  0%   37C    P5    34W / 250W |      0MiB / 11156MiB |      3%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
