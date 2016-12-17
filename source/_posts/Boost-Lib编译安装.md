---
title: Boost Lib编译安装
date: 2016-12-17 00:55:14
tags: C++
---
Boost是非常重要的c++库，提供各种丰富的功能。一直作为标准库的后备大营。
boost兼容linux，mac，windows三大操作系统。
在mac下使用boost，可以用homebrew安装，也可以编译安装，boost编译安装流程：

* 解压安装包
  
<code>tar -jxvf boost_1_62_0.tar.bz2

* 检查环境

```./bootstrap.sh```

* 编译

```./b2 -j 4```  4代表线程数，可以加快边缘时间

* 安装

```sudo ./b2 install ``` 一定要root权限才能install


![boost](https://p1.bqimg.com/567571/a9fdaf36b261f1a8.png)

  
  
	
