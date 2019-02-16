---
layout: post
date:   2019-2-15 00:00:00 -0900
title:  DockerでOpenVPNサーバを動かす
description: DockerへのOpenVPNサーバインストール方法など
categories: openvpn
tags:
- linux
- docker
- openvpn
---


### ホスト環境
```
$ cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)

$ docker --version
Docker version 1.13.1, build 8633870/1.13.1
$ docker-compose --version
docker-compose version 1.18.0, build 8dd22a9
```


### Dockerfile
公式のalpine3.9のイメージにopenvpnとeasyrsaをインストールするぐらい

```dockerfile
FROM alpine:3.9

RUN apk --update --no-cache --no-progress add \
        openvpn \
        easy-rsa \
        && \
    rm -rf /var/cache/apk/*

COPY run.sh /
CMD [ "/run.sh" ]
```


### サービス起動用のスクリプトrun.shを作成
下記のスクリプトを`run.sh`として作成する。  
`--push "route 192.168.1.0 255.255.255.0"`はOpenVPNサーバのネットワークに応じて変更する。

```sh
#!/bin/sh

# ERROR: Cannot open TUN/TAP dev /dev/net/tun: No such file or directory (errno=2)が出ないようにするため
mkdir -p /dev/net
if [ ! -c /dev/net/tun ]; then
    mknod /dev/net/tun c 10 200
fi

# クライアント用のネットワーク
OVPN_SERVER=${OVPN_SERVER:-10.8.0.0}

# `ip addr`コマンドの結果からネットワークデバイス名を抽出する
OVPN_NATDEVICE=$(ip addr | awk 'match($0, /global [[:alnum:]]+/) {print substr($0, RSTART+7, RLENGTH)}')
if [ -z "${OVPN_NATDEVICE}" ]; then
    ip addr
    echo "Failed to extract OVPN_NATDEVICE."
    exit 1
fi

# iptablesの設定
iptables -t nat -C POSTROUTING -s ${OVPN_SERVER}/24 -o ${OVPN_NATDEVICE} -j MASQUERADE || {
    iptables -t nat -A POSTROUTING -s ${OVPN_SERVER}/24 -o ${OVPN_NATDEVICE} -j MASQUERADE
}

# OpenVPNサーバの起動
/usr/sbin/openvpn \
    --cd ${OVPN_DIR:-/etc/openvpn} \
    \
    --port  ${OVPN_PORT:-1194} \
    --proto ${OVPN_PROTO:-udp4} \
    --dev   ${OVPN_DEV:-tun} \
    \
    --ca    ${OVPN_CA:-"/opt/easy-rsa/pki/ca.crt"} \
    --cert  ${OVPN_CERT:-"/opt/easy-rsa/pki/issued/server.crt"} \
    --key   ${OVPN_KEY:-"/opt/easy-rsa/pki/private/server.key"} \
    --dh    ${OVPN_DH:-"/opt/easy-rsa/pki/dh.pem"} \
    --tls-auth /opt/easy-rsa/pki/ta.key 0 \
    \
    --server ${OVPN_SERVER} ${OVPN_SERVER_MASK:-255.255.255.0} \
    --ifconfig-pool-persist ipp.txt \
    --push "redirect-gateway def1" \
    --push "route ${OVPN_SERVER} 255.255.0.0" \
    --push "route 192.168.1.0 255.255.255.0" \
    --push "dhcp-option DNS 8.8.8.8" \
    \
    --keepalive 10 120 \
    --user nobody \
    --group nobody \
    --persist-key \
    --persist-tun \
    \
    --status openvpn-status.log \
    --verb ${OVPN_VERB:-3} \
    \
    --compress lz4 \
    --tun-mtu 1500 \
    --mssfix 1460
```


### docker-compose.ymlの作成
- WANには1194番ポートはさらさずに54321番ポートでアクセスするようにする
    - WAN → ルータ54321 → Dockerホスト54321 → Dockerコンテナ1194  
- ホスト再起動時に自動起動するようにする（`restart: unless-stopped`）

```yaml
version: '3.2'
services:
  openvpn:
    build: .
    image: openvpn
    cap_add:
      - NET_ADMIN
    ports:
      - "54321:1194/udp"
    volumes:
      - ./easy-rsa:/opt/easy-rsa
    working_dir: /opt/easy-rsa
    environment:
      - OVPN_PORT=1194
    restart: unless-stopped
```


---
### OpenVPNサーバの設定
```shell-session
$ docker-compose build
$ docker-compose up -d
```
でコンテナを起動した状態で、マウントしている`/opt/easy-rsa`内でeasyrsaで証明書とかを生成する。

コンテナ内に入り、`/opt/easy-rsa`内に入り作業する。
```shell-session
$ docker-compose exec openvpn sh

# cd /opt/easy-rsa
```


#### PKIの初期化
```shell-session
# /usr/share/easy-rsa/easyrsa init-pki

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /opt/easy-rsa/pki
```

#### CA（認証局）構築
```shell-session
# /usr/share/easy-rsa/easyrsa build-ca nopass

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018
Generating RSA private key, 2048 bit long modulus (2 primes)
...............+++++
.........................+++++
e is 65537 (0x010001)
Can't load /opt/easy-rsa/pki/.rnd into RNG
140591198460776:error:2406F079:random number generator:RAND_load_file:Cannot open file:crypto/rand/randfile.c:98:Filename=/opt/easy-rsa/pki/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]: .

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/opt/easy-rsa/pki/ca.crt
```

#### Diffie-Helllmanパラメータの生成
```shell-session
# /usr/share/easy-rsa/easyrsa gen-dh

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
.............+.......
.....................
..........+..........
+....................
.....................
.......+.......+.....
DH parameters of size 2048 created at /opt/easy-rsa/pki/dh.pem
```


#### サーバー鍵の生成
```shell-session
# /usr/share/easy-rsa/easyrsa build-server-full server nopass

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018
Generating a RSA private key
...+++++
....+++++
writing new private key to '/opt/easy-rsa/pki/private/server.key.XXXXjcFHin'
-----
Using configuration from /usr/share/easy-rsa//safessl-easyrsa.cnf
Can't open /opt/easy-rsa/pki/index.txt.attr for reading, No such file or directory
140106217700200:error:02001002:system library:fopen:No such file or directory:crypto/bio/bss_file.c:72:fopen('/opt/easy-rsa/pki/index.txt.attr','r')
140106217700200:error:2006D080:BIO routines:BIO_new_file:no such file:crypto/bio/bss_file.c:79:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Jan 29 14:12:50 2022 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated
```


#### tls-auth用の鍵の生成
```sell-session
# openvpn --genkey --secret pki/ta.key
```


#### CRLファイルの生成
```shell-session
# /usr/share/easy-rsa/easyrsa gen-crl

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018
Using configuration from /usr/share/easy-rsa//safessl-easyrsa.cnf

An updated CRL has been created.
CRL file: /opt/easy-rsa/pki/crl.pem
```



---
#### クライアント鍵・設定ファイルを生成
`<CLIENT-NAME>`の部分にクライアント名に入れて実行する。

```shell-session
$ /usr/share/easy-rsa/easyrsa build-client-full <CLIENT-NAME> nopass

Using SSL: openssl OpenSSL 1.1.1a  20 Nov 2018
Generating a RSA private key
.................................................................................................+++++
..............................................................................+++++
writing new private key to '/opt/easy-rsa/pki/private/<CLIENT-NAME>.key.XXXXeDaKmJ'
-----
Using configuration from /usr/share/easy-rsa//safessl-easyrsa.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'<CLIENT-NAME>'
Certificate is to be certified until Jan 29 14:20:40 2022 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated
```

下記の内容で`<CLIENT-NAME>.ovpn`を作成する。  
`<SERVER-IP>`にはOpenVPNサーバのグローバルIPかホスト名を入れる。

```ovpn
client
dev tun
proto udp
remote <SERVER-IP> 54321
resolv-retry infinite
nobind
persist-key
persist-tun
auth-nocache
verb 3
key-direction 1
compress lz4
tun-mtu 1500
mssfix 1460
<ca>
  pki/ca.crtの内容をコピペ
</ca>
<cert>
  pki/issued/<CLIENT-NAME>.crtの内容をコピペ
</cert>
<key>
  pki/private/<CLIENT-NAME>.keyの内容をコピペ
</key>
<tls-auth>
  pki/ta.keyの内容をコピペ
</tls-auth>
```

作成したovpnをクライアントにインポートすれば接続できた。
