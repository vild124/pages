---
# layout: post
title:  "Raspberry Piとウェブカメラでmotionを使う"
date:   "2019-1-20 12:00:00"
categories: raspberrypi
tags:
- raspberrypi
---

## Raspberry Piとウェブカメラでmotionを使う

### motionをインストール
```shell-session
$ sudo apt-get -y install motion
```

### motionの設定
```conf
stream_localhost on
webcontrol_localhost on
```
を
```conf
stream_localhost off
webcontrol_localhost off
```
に変更する。


### 動作確認
USBカメラの確認
```shell-session
$ lsusb 
Bus 001 Device 005: ID 046d:082c Logitech, Inc. 
```

起動する
```shell-session
$ sudo motion -n
[0:motion] [NTC] [ALL] conf_load: Processing thread 0 - config file /etc/motion/motion.conf
[0:motion] [NTC] [ALL] motion_startup: Motion 4.0 Started
[0:motion] [NTC] [ALL] create_path: creating directory /var/log/motion
[0:motion] [NTC] [ALL] motion_startup: Logging to file (/var/log/motion/motion.log)
```

`Ctrl+C`で止めて、
`/var/lib/motion`に画像が保存るはずなのでsftp等で確認する。

