---
title: '关于C++返回值问题的思考'
date: 2017-1-12 14:13:20
tags: C++
---

在目前很多C++的教程书籍中，关于函数返回值问题，一直以来，没有一个比较让人信服的解释。国内的C++专家不在少数，但是极少看到有人在百度能搜索到的网站上把这个问题解释清楚的。

接下来我来说说我自己对这个问题的思考。首先，C++返回值问题牵涉到一个很重要的概念，叫*返回值优化（NRV）*，或许你知道NRV之后，在去查询相关资料这个问题就很容易理解了，但是在刚入门阶段，很多人是不知道NRV，同时对编译器做的手脚也无从感知。
<!--more-->

其实，在真正的编译器转化后的代码中并没有返回函数这个类型。所谓的返回值还是通过传递引用和辅助一些临时变量来实现的。NRV技术曾经也在C++标准委员会内部发生过很大的争论。目前，这个技术应该是各大主流编译器必须支持的东西了。

接下来看如下代码：

```cpp
#include<iostream>
#include <cstring>

class elem{
public:
    elem(){
        std::cout<<"CONSTRUCT"<<std::endl;
        num = 0;
    }
    elem(int x):num(x){
        std::cout<<"CONSTRUCT BY INT"<<std::endl;
    }
    elem(const elem &src){
        std::cout<<"COPY"<<std::endl;
        memcpy(this,&src,sizeof(elem));
    }
    elem &operator=(const elem &src){
        std::cout<<"EQUALL"<<std::endl;
        memcpy(this,&src,sizeof(elem));
        return *this;
    }

private:
    int num;
};

elem test()
{
    elem one(2);
    return one;
}


int main()
{
    elem two = test();
    return 0;
}
```

以上代码执行的结果就很有趣，它输出CONSTRUCT BY INT，也就是说它只执行了一次elem(int x)构造函数。很多人看到这里会觉得很懵，太不科学了。
实际上这段代码在编译里面转化以后，如下：

```cpp
void test(elem & __return)
{

}

int main()
{

elem two(2);
test(two);

return 0;
}

```

编译器忽略了局部变量one，这就是NRV技术的本质，就是为了减少内存的拷贝问题。如果对象比较小，或许拷贝的代价不是很大，但是如果是一个很大的对象，或者，调用次数频繁，必然会带来很大的性能折扣。因此，NRV技术可以有效的避免这种情况，提高程序效率。

OK，我们再来看一种场景，如果是以下代码的情况又该如何呢？

```cpp
#include<iostream>
#include <cstring>

class elem{
public:
    elem(){
        std::cout<<"CONSTRUCT"<<std::endl;
        num = 0;
    }
    elem(int x):num(x){
        std::cout<<"CONSTRUCT BY INT"<<std::endl;
    }
    elem(const elem &src){
        std::cout<<"COPY"<<std::endl;
        memcpy(this,&src,sizeof(elem));
    }
    elem &operator=(const elem &src){
        std::cout<<"EQUALL"<<std::endl;
        memcpy(this,&src,sizeof(elem));
        return *this;
    }
    void printnum(){
        std::cout<<num<<std::endl;
    }

private:
    int num;
};

elem test()
{
    elem one(2);
    return one;
}


int main()
{
    test().printnum();
    return 0;
}
```

以上代码在编译器转换的过程中会生成一个局部变量，通过局部变量来实现。
转换以后如下：

```cpp
void test(elem & __return )
{
   
}


int main()
{
	elem __temp(2);
	test(__temp);
    	__temp.printnum();
    	return 0;
}
```







