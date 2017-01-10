---
title: 'compare_exchange_weak(strong)详解'
date: 2017-1-10 22:00:20
tags: Words
---
这段时间在学习《C++并发编程实践>>，在这里要吐槽一下这本书的翻译者，翻译真是烂。我觉得译者本人很多地方自己都没理解。比如这个原子操作中很关键的一个函数compare_exchange_weak和compare_exchange_strong函数，译者就没有说清楚这个函数的本质。
<!--more-->
这两个函数weak和strong的区别就在于weak在一些平台可能会出现伪失败的情况，strong就不会，但是weak效率更高。intel目前的主流CPU在ubuntu上不会出现伪装失败的情况。
如果担心出现weak出现伪失败，可以搭配以下while循环使用

```cpp
bool expected = false;
std::atomic<bool> b(true);
while(!b.compare_exchange_weak(expected,true)&&!expected)
```


```cpp
 bool expected = true;
    std::atomic<bool> b(false);
    std::cout<<b.compare_exchange_weak(expected,true)<<" ";

    std::cout<<expected<<" "<<b.load()<<std::endl;
```
以上代码的执行的逻辑应该是(无论strong或者weak)

如果b和expected相等.那么,b会被修改为compare_exchange_weak()的第二个参数,然后函数的返回值为true.
如果b和expected不相等,那么b会被修改为expected的值.同时函数返回为false.



