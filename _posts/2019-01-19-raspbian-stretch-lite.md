---
title:  "Raspbian Stretchをディスプレイなしでインストールする"
date:   2019-1-19 19:00:00 -0700
categories: raspberrypi
tags:
- raspberrypi
---

## Raspbianをディスプレイなしでインストールする

### Raspbianをダウンロード
[Download Raspbian for Raspberry Pi](https://www.raspberrypi.org/downloads/raspbian/)から  
Raspbian Stretch Lite (`2018-11-13-raspbian-stretch-lite.img`)をダウンロードする


### Raspbianの書き込み
`dd`で書き込む

```bash
$ sudo fdisk -l
...
ディスク /dev/sdc: 14.5 GiB, 15548284928 バイト, 30367744 セクタ
ディスク型式: Flash Reader    
単位: セクタ (1 * 512 = 512 バイト)
セクタサイズ (論理 / 物理): 512 バイト / 512 バイト
I/O サイズ (最小 / 推奨): 512 バイト / 512 バイト
ディスクラベルのタイプ: dos
ディスク識別子: 0x00000000

デバイス   起動 開始位置 終了位置   セクタ サイズ Id タイプ
/dev/sdc1  *        2048 30367743 30365696  14.5G  c W95 FAT32 (LBA)


$ sudo dd bs=64MB if=2018-11-13-raspbian-stretch-lite.img of=/dev/sdc
29+1 レコード入力
29+1 レコード出力
1866465280 bytes (1.9 GB, 1.7 GiB) copied, 411.178 s, 4.5 MB/s
```


### sshd有効化
デフォルトでsshdが自動機能しないためSSHログインはできない。

[How to enable SSH on Raspbian without a screen - howchoo](https://howchoo.com/g/ote0ywmzywj/how-to-enable-ssh-on-raspbian-without-a-screen)  
の通り下記コマンドを試したが`ssh: connect to host 192.168.1.17 port 22: Connection refused`で接続できなかった。
```
$ sudo fdisk -l
デバイス   起動 開始位置 終了位置   セクタ サイズ Id タイプ
/dev/sdc1           8192    98045    89854  43.9M  c W95 FAT32 (LBA)
/dev/sdc2          98304 30367743 30269440  14.4G 83 Linux

$ sudo mount /dev/sdc2 /mnt/pi
$ cd /mnt/pi/boot
$ sudo touch ssh
```

諦めてキーボード、ディスプレイを接続して下記コマンドで有効化する。
```bash
$ sudo systemctl start sshd
$ sudo systemctl enable sshd
```


### SSHでログイン
ルータのDHCP割り当て状態からRaspberry PiのMACアドレス（`B8:27:EB:??:??:??`）に割り当てられているIPを調べる

Yamaha NVR510の場合は`show status dhcp`でホスト名まで表示された
```
  Leased address: 192.168.1.17
(type) Client ID: (01) b8 27 eb 60 59 21
       Host Name: raspberrypi
 Remaining lease: 2days 23hours 59min. 31secs.
```

下記ユーザ名・パスワードでSSHログインする  
- ユーザー名: pi
- パスワード: raspberry


---
### その他の設定
#### IP固定
インターフェース名の確認
```bash
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:xx:xx:xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.17/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::11cb:3d7e:78f0:2ead/64 scope link 
       valid_lft forever preferred_lft forever

```

`/etc/dhcpcd.conf`の編集
```bash
interface eth0
static ip_address=192.168.1.27/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```
再起動
```
$ sudo reboot
```
