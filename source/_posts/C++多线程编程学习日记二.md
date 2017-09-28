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


```c++
//created by mayusheng
#include <boost/thread/shared_mutex.hpp>
std::shared_mutex s_mutex;
boost::shared_lock<boost::shared_mutex> lk(s_mutex);
//读者
std::lock_guard<boost::shared_mutex> lk(s_mutex);
//写者
```


