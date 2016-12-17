---
title: 在ubuntu下编译安装unbound,DNS秒解析
date: 2016-12-17 23:40:00
tags: Linux
---
unbound是一款可以媲美BIND的DNS服务软件。它配置简单，稳定，高效，而且非常灵活！

首先，你需要一个Ubuntu操作系统的主机，如果作为局域网使用，那么，主机具有局域网IP即可。如果要做public DNS，那么需要有公网IP，或者通过路由器做端口转发！
<!--more-->

通过SSH连接到你的主机，按照惯例，先来一波update！

(本文指令皆以root权限进行，如果没有root账号权限，请sudo）

``` 
apt-get update 
```

接下来，我们其实就可以安装Ubuntu软件源提供的unbound，但是为了提高性能，我们决定自己编译安装，这样既能用上最新版本，同时，还可以在编译时候设置一些参数，来提高性能！

接下来，配置unbound编译环境，借助强大的apt-get，我们一个指令就可以搞定！

```
apt-get build-dep unbound
```

执行过程中，你可以有空闲去喝杯咖啡！

一切进行顺利的话，我们就可以开始获取unbound最新包了！

```
wget http://www.unbound.net/downloads/unbound-latest.tar.gz
```
 

然后解压该文件

```
tar zxvf unbound-latest.tar.gz 
```

ok接下来进入unbond-latest文件目录，执行以下指令

``` 
cd unbound-latest 
```

``` 
./configure --prefix=/usr --sysconfdir=/etc --disable-rpath --with-pidfile=/var/run/unbound.pid --with-libevent 
```

libevent是我们最需要的，因为它可以极大的提高unbound事件处理性能！

接下来执行安装

```
make && make install 
```

执行完毕，主机就已经安装了unbound了。先不要着急启动它，我们还需要配置文件

首先，执行以下指令

``` 
unbound-control-setup 
```

生成控制密钥

接下来进入unbound配置目录

``` 
cd /etc/unbound 
```

在此，本人祭出我自己的配置文件，大家可供参考！

```
server:
verbosity: 1
statistics-interval: 0
statistics-cumulative: no
extended-statistics: yes
num-threads: 2 //线程数
msg-cache-slabs: 2 //此处最好和线程数相同
rrset-cache-slabs: 2
infra-cache-slabs: 2
key-cache-slabs: 2
interface: 0.0.0.0 //监听IP
port: 53 //端口号
do-ip4: yes
do-ip6: no
do-udp: yes //支持UDP协议查询
do-tcp: yes //支持TCP协议查询
tcp-upstream: yes //强制和上游DNS用TCP通讯，前提是上游DNS要支持TCP
outgoing-num-tcp: 64 //出口连接数
interface-automatic: no
access-control: 10.0.3.0/24 allow //运行使用该服务器的主机，如果要给所有IP
access-control: 127.0.0.1 allow//使用请用0.0.0.0/0
chroot: ""
username: "unbound" //用户名
directory: "/usr/local/etc/unbound"
log-time-ascii: yes
pidfile: "/var/run/unbound.pid"
harden-glue: yes
harden-dnssec-stripped: yes
harden-below-nxdomain: yes
harden-referral-path: yes
unwanted-reply-threshold: 10000000
prefetch: yes
prefetch-key: yes
rrset-roundrobin: yes
minimal-responses: yes
val-clean-additional: yes
val-permissive-mode: no
val-log-level: 1
rrset-cache-size: 200m
msg-cache-size: 100m
outgoing-range: 4096
num-queries-per-thread: 2048
so-rcvbuf: 8m
so-sndbuf: 8m
so-reuseport: yes
cache-max-ttl: 604800
cache-min-ttl: 432000

remote-control: //远程控制
control-enable: yes
server-key-file: "/usr/local/etc/unbound/unbound_server.key"
server-cert-file: "/usr/local/etc/unbound/unbound_server.pem"
control-key-file: "/usr/local/etc/unbound/unbound_control.key"
control-cert-file: "/usr/local/etc/unbound/unbound_control.pem"

include: "/usr/local/etc/unbound/unbound.forward-zone.China.conf"
forward-zone:
name: .
forward-addr: 8.8.8.8 //上层DNS服务器IP
forward-addr: 8.8.4.4
forward-addr: 64.6.64.6
forward-addr: 64.6.65.6
forward-addr: 208.67.222.222
forward-addr: 208.67.220.220
forward-addr: 208.67.222.220
forward-addr: 208.67.220.222
```

可以根据我的配置文件进行一些适当的调整，以符合自己的需求。如果不需要DNS抗污染的话，可以把上层DNS的IP换成国内的DNS，比如223.5.5.5（阿里）、114.114.114.114（114）、1.2.4.8（中国互联网中心），不过国内这些都不支持TCP协议，因此需要关闭tcp-upstream: no。

还不要着急运行unbound，我们还需要添加unbound用户。

``` 
groupadd unbound 
```

``` 
useradd -m -g unbound -s /bin/false unbound 
```

OK，接下来，还可以借助unbound-checkconf检查一下配置文件。

``` 
unbound-checkconf 
```

如果一切无误。就可以启动unbound啦！

``` 
unbound 
```

如果没有报错，就可以用上unbound作为本地dns了，如果在局域网中做dns中，可以感受到dns缓存所带来的速度提升感，在UNIX系统中，可以用dig指令来检查部署效果。以后打开每一个网页，可以节约40ms，是不是感觉生命得到了更加充分的利用？

``` 
dig www.baidu.com @your-dns-ip 
```

你可以发现第一次差不多在40ms左右，第二次，一般就在0~10ms左右，此时的速度就取决去你服务器性能和本地局域网性能了！
共享一下自己的***DNS:115.159.39.228 TCP&UDP***

谢谢！
