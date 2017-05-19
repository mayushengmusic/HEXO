---
title: 'C++多线程编程学习日记二'
date: 2017-5-19 12:13:20
tags: C++
---

C++11非常让人着迷，记录这段时间学习的C++11多线程库的新技巧。
<!--more-->

* std::call_once的使用

在实际编程的过程中，经常需要多个进程去竞争一个代码段的执行权利。也就是说，某一段代码只能一个进程执行它，而且只能执行一次。在设计的时候，又很难确认让具体的进程去执行。比如，要实例化一个对象*test_class*,然后去调用*test_class.show()*函数，show()函数本质是一个只读函数，因此可以多过线程共享一个对象，然后调用。但是初始化对象的代码只能执行一次。

对于这种应用场景，C++ 11提供了call_once的工具来实现。

样例代码如下：

```cpp
class test_class{
public:
    test_class(const std::string & greeter):greeter(greeter){
            std::cout<<"construct"<<std::endl;
    }
    void show(){
        std::cout<<greeter<<std::endl;
    }

public:
    std::string greeter;
};

void setptr(std::unique_ptr<test_class> & src){
    src=std::make_unique<test_class>(test_class("hello,world"));
}

std::unique_ptr<test_class> ptr;
std::once_flag init_flag;

void work(){
    std::call_once(init_flag,setptr,std::ref(ptr));
    ptr->show();
}


int main()
{

    std::thread t1(work);
    std::thread t2(work);
    t1.join();
    t2.join();
    
    return 0;
}
```

以上代码在ptr的初始化中，只是执行了一次，对于传入的std::call_once传入参数，需要注意，函数必须传入引用。可以看到结果输出构造函数只执行了一次。std::call_once如果绑定类的成员函数，有一个需求无法解决，绑定重载的函数（同名不同参），无法进行绑定，这个和std::bind一样。

* 读写互斥元，BOOST库实现

在C++中遇到读者写者问题，如果只是使用标准库的mutex互斥元会导致非常严重的性能损失。读者写者问题，读者之间并不存在竞争，对数据无需进行保护，但是，如果写者一旦有动作，就必须独占数据，所有读者不能读取，其他写者也无法写入。
读者方面(这里只有锁定方法）

```cpp
#include <boost/thread/shared_mutex.hpp>
std::shared_mutex s_mutex;
boost::shared_lock<boost::shared_mutex> lk(s_mutex);
```

写者方面(同上）

```cpp
std::lock_guard<boost::shared_mutex> lk(s_mutex);
``` 

* 线程安全队列的实现

队列在单线程的运行环境中，我们常用的接口，包括empty，pop，front等等，在多线程的时候，这些接口之间其实存在固有的竞争关系。比如线程A调用front拿到了队列头数据，下一秒它准备pop的时候，或许线程B已经抢先一步pop了队列。还有线程A检查了队列此刻不为空，或许下一秒它准备取数据的时候，线程B已经把队列取空了。因此，按照常用的接口来设计线程安全队列是不可靠的。

在线程安全队列的实现，我们采用C++11的条件变量来实现堵塞模式。实现代码如下：

```cpp
#include <mutex>
#include <iostream>
#include <condition_variable>
#include <queue>


template<typename T>
class threadsafe_queue{
public:
    threadsafe_queue();
    threadsafe_queue(const threadsafe_queue &);
    threadsafe_queue &operator=(const threadsafe_queue &) = delete;
    void push(T new_value);
    bool try_pop(T & value);
    std::shared_ptr<T> try_pop();

    void wait_and_pop(T & value);
    std::shared_ptr<T> wait_and_pop();
    bool empty() const;

private:
    mutable std::mutex mut;
    std::queue<T> data_queue;
    std::condition_variable data_cond;
};

template <typename T>
void threadsafe_queue<T>::push(T new_value) {
    std::lock_guard<std::mutex> lk(mut);
    data_queue.push(new_value);
    data_cond.notify_one();
}

template <typename T>
void threadsafe_queue<T>::wait_and_pop(T &value) {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk,[this]{
        return !this->data_queue.empty();
    });
    value=data_queue.front();
    data_queue.pop();
}

template <typename T>
bool threadsafe_queue<T>::try_pop(T &value) {
    std::lock_guard<std::mutex> lk(mut);
    if(!data_queue.empty())
        return false;
    else
    {
        value=data_queue.front();
        data_queue.pop();
        return true;
    }
}

template <typename T>
std::shared_ptr<T> threadsafe_queue<T>::wait_and_pop() {
    std::unique_lock<std::mutex> lk(mut);
    data_cond.wait(lk,[this]{
        return !this->data_queue.empty();
    });
    std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
    data_queue.pop();
    return res;
}

template <typename T>
std::shared_ptr<T> threadsafe_queue<T>::try_pop() {
    std::lock_guard<std::mutex> lk(mut);
    if(!data_queue.empty())
        return std::shared_ptr<T>();
    std::shared_ptr<T> res(std::make_shared<T>(data_queue.front()));
    data_queue.pop();
    return res;
}

template <typename T>
bool threadsafe_queue<T>::empty() const {
    std::lock_guard<std::mutex> lk(mut);
    return data_queue.empty();
}

template <typename T>
threadsafe_queue<T>::threadsafe_queue() {
}

template <typename T>
threadsafe_queue<T>::threadsafe_queue(const threadsafe_queue &other) {
    std::lock_guard<std::mutex> lk(other.mut);
    data_queue=other.data_queue;
}




int main()
{
    threadsafe_queue<int> queue;
    std::thread t1([&queue]{std::cout<<*queue.wait_and_pop()<<std::endl;});
    std::thread t2([&queue]{std::this_thread::sleep_for(std::chrono::seconds(3));queue.push(3);});
    t2.join();
    t1.join();

    return 0;
}
```
条件变量 *data_cond.notify_one()*，可以尝试唤醒等待进程队列中的进程，从而进行队列中的数据处理。

```cpp
data_cond.wait(lk,[this]{
        return !this->data_queue.empty();
    });
```
    
以上代码，如果队列为空，那么执行到这里的进程会进入等待状态，直到notify_one()提供一个提醒，那么才会继续检查队列，直到不为空，才会继续下面的运行，在检查的过程中，如果队列为空，会释放手里的互斥量，如果不为空，会继续锁住，进行下面的操作。这里对队列的检查函数写在lamdba表达式中，其实这里，会有多次伪唤醒，具体次数由平台决定，在ubuntu上测试，会有一次伪唤醒，代码第一次执行到此处，会主动检查队列。


