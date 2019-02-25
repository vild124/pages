---
layout: post
date:   2019-2-25 15:00:00 +0900
title:  LinuxでHDDのパーティションテーブル作成とフォーマット方法
description: Ubuntu18.04にHDDを追加した時のHDDのフォーマット作業のメモ
categories: ubuntu
tags:
- linux
- ubuntu
---


Ubuntu18.04にHDDを追加した時のHDDのフォーマット方法。  
多分他のディストリビューションでも同じはず。


---
### パーティションテーブルの作成
パーティションテーブルの作成はfdiskとpartedでできる。

- `fdisk`は2TB未満のパーティションのみ作成可能
- `parted`は2TB以上のパーティションも作成可能

なので基本的にはpartedを使えば良い。


#### fdiskでのパーティションテーブル作成方法
一応fdiskの使い方もメモ。

ディスクの確認を行う
```shell-session
$ sudo fdisk -l | grep "Disk /dev/"
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Disk /dev/sdb: 223.6 GiB, 240057409536 bytes, 468862128 sectors
```
今回は`/dev/sda`にパーティションテーブルを作成する 
```shell-session
$ sudo fdisk /dev/sda
```

まず，dコマンドで不要なパーティションを削除する。  
パーティションがなければ`No partition is defined yet!`と出力される  
```shell-session
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): d
No partition is defined yet!
Could not delete partition 93935481966361
```

nコマンドで新規パーティションを作成する。  
ストレージとして用いる場合はすべてデフォルト（ENTERキー連打）で全ディスクスペースを使用して1つのパーティションを作れる。
```shell-session
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-1953525167, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-1953525167, default 1953525167):

Created a new partition 1 of type 'Linux' and of size 931.5 GiB.
```

wコマンドでパーティションテーブルを書き込む。
```shell-session
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```


### partedでのパーティションテーブル作成方法
ディスクを確認する。
```shell-session
$ sudo parted -l | grep "Disk /dev/"
Disk /dev/sda: 1000GB
Disk /dev/sdb: 240GB
```
今回は`/dev/sda`にパーティションテーブルを作成する。
```shell-session
$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) 
```

ディスクラベルを設定する。
```shell-session
(parted) mklabel                                        
New disk label type? gpt
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will be lost. Do you
want to continue?
Yes/No? Y
```
ディスクラベルの設定は下記1コマンドでも同じようにできる。
```shell-session
(parted) mklabel gpt Y
```

新規パーティションを作成する。  
パーティション名は設定しないため，空エンター
```shell-session
(parted) mkpart
Partition name?  []?
File system type?  [ext2]? ext4
Start? 0%
End? 100%
```
ディスクラベルの設定の時と同じように1コマンドでもOK。
```shell-session
(parted) mkpart ext4 0% 100%
```

partedの終了
```shell-session                                                                
(parted) quit
```

---
### ファイルシステムの作成
fdisk・partedのどちらを使用した場合でもext4でフォーマットする場合は以下のコマンドで実行できる。
```shell-session
$ sudo mkfs -t ext4 /dev/sda1
```
