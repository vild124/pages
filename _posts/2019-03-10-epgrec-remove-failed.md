---
layout: post
title:  epgrecで失敗した予約録画の削除方法
description: epgrecの録画済一覧で削除できないデータを削除する方法
categories: linux
tags:
- linux
- epgrec
---


### データベースに接続する
```shell-session
$ mysql -u ユーザー名 -p
Enter password:
```

### データベースの切り替え
```sql
> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| epgrec             |
+--------------------+

> use epgrec;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
```

### 録画IDの確認
```sql
> select id, title from Recorder_reserveTbl where complete = 0 and endtime < now();
+----+-----------------+
| id | title           |
+----+-----------------+
| 28 | 録画タイトル1   |
| 36 | 録画タイトル2   |
+----+-----------------+
```

### 録画済みフラグを立てる
```sql
> update Recorder_reserveTbl set complete = 1 where id = 録画id;
Query OK, 1 row affected (0.00 sec)
```

### 終了
```sql
> exit;
```

これでepgrec上で手動で録画済一覧から削除できるようになる
