---
title: 'ubuntu下crontab执行出错的解决方案'
date: 2016-12-19 00:48:12
tags: Linux
---

前几天在ubuntu下设置root账户的crontab，设置完成以后死活不执行，让我非常恼火。经过一番检查，发现是环境变量没有设置对，crontab需要手动设置环境变量，要不然那些什么/usr/sbin /usr/local/bin等等统统出错。
![ubuntu](https://p1.bqimg.com/567571/9a5f4445a875c1af.png)
<!--more-->

要解决这个问题，其实很好办。

* 在执行完成

```
crontab -u root -e
```
* 进入设置以后，需要在最前面加上一行

```
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
```
有了这一行的加持，一切就完美解决了！