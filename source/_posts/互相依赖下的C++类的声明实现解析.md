---
title: '互相依赖下的C++类的声明实现解析'
date: 2016-12-20 17:19:20
tags: C++
---

今天看《C++高级编程》看到友元相关的问题，友元类（friend class)很容易理解，在编码上也没有太大难度，由于友元类是全局的，也就是说如果A类是B类的友元类，那么，B类中的所有private、protected成员变量全部暴露在A的成员函数中，完全可以调用。当然，这种方案可能会导致严重的安全问题。
<!--more-->
C++引入了成员函数友元，只定义A类中的某个成员函数做B的友元，那么，A类中，除了这个成员函数，其他的都无法访问B的private、protected成员变量。但是，书上的代码只是寥寥几行，这个部分，作者并没有说明白，正常思维下，这个编码是无法通过的。因为涉及到A类中调用B类，B类中又涉及A类。
这是一种强耦合关系，必须依赖C++的前置声明来解决。现在，很多开源项目开始广泛使用.hpp头文件，如果使用.hpp头文件，目前应该无法解决这种强耦合关系。如果习惯使用.hpp那么尽量不要用这种强耦合模型。
首先，先看糅合在一个文件里面的版本。

*main.c*

```cc
#include "APP/SpreadsheetCell.hpp"
class A;//前置声明A
class B{
public:
B();
~B();
void setBNumber(A &x);
int getBNumber();
private:
A *Number;//只有定义为指针或者引用才可以用前置声明
};
class A{
public:
friend void B::setBNumber(A &x);//B类的setNumber函数是A的友元函数
A();
void setANumber(int x);
int getANumber();
private:
int Number;
};
A::A() {
Number = 0;
}
void A::setANumber(int x) {
Number = x;
}
int A::getANumber() {
return Number;
}
B::B() {
Number = new A;
}
B::~B() {
delete Number;
}
void B::setBNumber(A &x) {
Number->Number=x.Number;
}
int B::getBNumber() {
return Number->getANumber();
}
auto main()->int {
B b;
A a;
a.setANumber(5);
b.setBNumber(a);
cout<<b.getBNumber()<<endl;
return 0;
}
```
A类声明B类的成员函数为它的友元的时候，B类的定义一定要在前面。
B类中使用A类，只能是先给个空壳，毕竟A类的具体实现还没有，因为声明的顺序必须B在A前面，但是可以用前置声明A
这里说的空壳，就是B此时使用A，只能用指向A的指针以及A的引用。
OK，接下来，B类开始实现的时候，就可以肆无忌惮的使用A具体方法，因为此时A的声明已经在B类的实现之前。
写在一个文件里面还是很清爽的，逻辑上也很好理解，但是要拆分到5个文件的时候，我一开始还是挺懵逼的。这里说的五个文件为A.h、A.cpp、B.h、B.cpp main.cpp。
接下来，贴出成功拆分以后的实现代码：

*A.h*

```cc
#ifndef C_PRIMARY_A_H
#define C_PRIMARY_A_H
#include "B.h"
class A{
public:
friend void B::setBNumber(A &x);
A();
void setANumber(int x);
int getANumber();
private:
int Number;
};
#endif //C_PRIMARY_A_H
A.h中声明B类的setBNumber方法为A的友元，因此，必须包含B.h
A.cpp
#include "A.h"
A::A() {
Number = 0;
}
void A::setANumber(int x) {
Number = x;
}
int A::getANumber() {
return Number;
}
A.cpp就是A类的具体实现，这里很好理解。
B.h
#ifndef C_PRIMARY_B_H
#define C_PRIMARY_B_H
class A;
class B{
public:
B();
~B();
void setBNumber(A &x);
int getBNumber();
private:
A *Number;
};
```
这里B.h里面采用了一个A类的前置声明

*B.cpp*

```cc
#include "A.h"
#include "B.h"
B::B() {
Number = new A;
}
B::~B() {
delete Number;
}
void B::setBNumber(A &x) {
Number->Number=x.Number;
}
int B::getBNumber() {
return Number->getANumber();
}
由于B类的具体实现中，必须用到A类的方法，因此要包含A.h
最后是main.c(测试用）
#include "STUDY/A.h"
#include "STUDY/B.h"
#include
using namespace std;
auto main()->int {
B b;
A a;
a.setANumber(5);
b.setBNumber(a);
cout<<b.getBNumber()<<endl;
return 0;
}
```

这里调用就和普通的类调用一样，只是A、B类编译的时候，需要注意以上一些问题。不过对于友元类还是尽量少用，一来容易出问题，遇到这种强耦合关系很费事。二来，友元类不太符合标准接口的规范。
