---
title: 'share_future和future的对比'
date: 2016-12-23 16:43:20
tags: C++
---
future和promise搭配可以很方便的实现在多个进程中传递数据,由于future本身不可复制,只能*移动*,因此当在多个进程中等待promise的数据的时候,future就会出现问题.因为它只能传递数据给一个进程,这里就产生了竞争.
<!--more-->

如何避免这种竞争呢?那就使用share_future,share_future是可*复制*的.因此,通过future生成share_future以后可以通过share_future传递多个值给多个进程的同时不会产生竞争.

测试代码如下:

```cpp
#include <future>
#include <iostream>
#include <string>

int main()
{
    std::promise<std::string> p;

    std::future<std::string> f(p.get_future());
    std::thread t1([&f](){
        std::cout<<"t1: "<<f.get()<<std::endl;
        ;});
    std::thread t2([&f](){
        std::cout<<"t2: "<<f.get()<<std::endl;
        ;});
    std::thread t3([&f](){
        std::cout<<"t3: "<<f.get()<<std::endl;
        ;});

    std::this_thread::sleep_for(std::chrono::seconds(5));
    p.set_value("hello,son!");
    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

这段代码的运行结果,可以看到只有一个进程拿到了hello,son字符串.

```cpp
#include <future>
#include <iostream>
#include <string>


int main()
{
    std::promise<std::string> p;

    std::shared_future<std::string> sf(p.get_future());
    std::thread t1([sf](){
        std::cout<<"t1: "<<sf.get()<<std::endl;
        ;});
    std::thread t2([sf](){
        std::cout<<"t2: "<<sf.get()<<std::endl;
        ;});
    std::thread t3([sf](){
        std::cout<<"t3: "<<sf.get()<<std::endl;
        ;});

    std::this_thread::sleep_for(std::chrono::seconds(5));
    p.set_value("hello,son!");
    t1.join();
    t2.join();
    t3.join();
    return 0;
}
```

以上代码执行以后可以看到三个进程都拿到了字符串,需要主要lamdba表达式中的拷贝和引用的区别.