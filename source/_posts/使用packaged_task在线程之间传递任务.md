---
title: '使用packaged_task在线程之间传递任务'
date: 2016-12-21 16:21:20
tags: C++
---

C++11标准引入了很多非常好的多线程支持工具,条件变量提供了事件驱动C++多线程模型,可以一定程度上摆脱难用的C语言事件库,比如libevent等等.这里提供了std::packaged这款工具,更是让C++多线程的灵活度提高了一个level,学习完以后,可以自己实现简单的线程池了!
![多线程](https://i1.piimg.com/567571/94b9b31b5565c41f.jpg)
<!--more-->

std::packaged_task()是把std::future封装到可调用对象上.我写了一个简单的两个线程之间传递任务的例子.

* code

```cpp
#include <deque>
#include <mutex>
#include <future>
#include <thread>
#include <utility>
#include <iostream>
#include <vector>

std::mutex m;
std::deque<std::packaged_task<void(int)>> tasks;
int timex=100;

std::future<void> productor(std::function<void(int)> f)
{
    std::packaged_task<void(int)> task(f);
    std::future<void> res=task.get_future();
    std::lock_guard<std::mutex> lk(m);
    tasks.push_back(std::move(task));
    return res;
}

void threadconsumer()
{
    int a=0;
    std::packaged_task<void(int)> task;
    while(true)
    {
        std::lock_guard<std::mutex> lk(m);
        if(tasks.empty())
            continue;
        task = std::move(tasks.front());
        tasks.pop_front();
        a++;
        std::this_thread::sleep_for(std::chrono::seconds(5));
        task(a);
        if(a==timex)
            break;
    }

}

void threadproduct()
{
    auto func=[](int x){
        std::cout<<" task packets "<<x<<std::endl;
    };
    for(int i=0;i<timex;i++)
    {
        auto futurex = productor(func);
        futurex.wait();
        std::cout<<"wait....\n";
    }
}



int main()
{
    std::thread t1(threadproduct);
    std::thread t2(threadconsumer);
    t1.join();
    t2.join();

    return 0;

}
```

线程t1封装了一个lambda表达式任务给队列,t2线程在队列里面取出任务,并执行.这里的例子略显得无聊,这种任务其实仿函数也能做.但是为什么还要引进这个工具呢.注意观察futurex.wait(),通过future,t1线程可以等待t2线程中任务的执行,future还提供了调用返回值的函数get(),因此,可以实现创建一个任务在B线程中执行,执行完B线程获取返回值.这种机制非常类似进程池的模型.
