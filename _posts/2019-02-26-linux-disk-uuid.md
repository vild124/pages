---
layout: post
title:  LinuxでUUIDでHDDをマウントする
description: UbuntuとCentOS上でのUUID確認と/etc/fstabでの自動マウント方法
categories: linux
tags:
- ubuntu
- centos
---

### UUIDの確認
CentOS7でもUbuntu18.04でも、blkidの出力をgrepすればUUIDを確認できる。
```shell-session
$ sudo blkid | grep /dev/sdd1
```

CentOSの場合は`UUID="..."`の部分が必要
```shell-session
$ sudo blkid | grep /dev/sda1
/dev/sda1: UUID="eda3624c-15a5-4871-931e-5fbcaedcdfac" TYPE="ext4"
```
Ubuntuの場合は`PARTUUID="..."`の部分が必要
```shell-session
$ sudo blkid | grep /dev/sdd1
/dev/sdd1: PARTLABEL="ext4" PARTUUID="dd120623-d167-43d5-a372-ba77b4b4f15c"
```


### コマンドでマウントする
`UUID="..."`や`PARTUUID="..."`の部分をそのまま使用して
```shell-session
$ sudo mount -t ext4 UUID="..." /mnt
```
とか
```shell-session
$ sudo mount -t ext4 PARTUUID="..." /mnt
```
でマウントできる


### 自動マウント
自動マウントする場合は/etc/fstabに`UUID="..."`や`PARTUUID="..."`の部分をそのまま使って書く。

```shell-session
$ sudo vim /etc/fstab
UUID="..."     /mnt/hdd    ext4    defaults    0 0
PARTUUID="..." /mnt/hdd    ext4    defaults    0 0
```
