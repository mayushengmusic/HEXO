---
title: 'C++多线程编程学习日记一'
date: 2017-5-5 11:04:20
tags: C++
---

经历了一个多月的找实习经历，顺利拿到offer，终于可以安静的学习一段时间。多线程编程一直是程序员必备技能之一。但是，多线程编程是一个非常考验功力的领域。C++11在标准库中引入了多线程支持，提供了很方便的多线程实现手段。在这里写下学习中的问题和知识点。

<!--more-->

线程之间难免需要进行数据共享。然而数据的共享就会遇到竞争，如果没有线程需要对数据进行修改，那么这种竞争是无害的，不过，很多情况下线程需要对共享数据进行修改，很多时候，修改会破坏一些*不变量*（数据结构上约定的规则），比如，单向链表，在进行节点删除的过程中，会出现断裂的情况。如果不对数据进行保护，那么，其他的线程会读取到一个错误的单向链表。

如何对共享数据进行保护措施，最容易想到的方案就是互斥量，在C++11中，提供了非常灵活的互斥量解决方案。

* std::mutex

  std::mutex 提供了一个非常简单的互斥量，可以通过调用lock()和unlock()函数进行锁定和解锁。对一个临界区，可以采用一个std::mutex进行保护。在进入临界区之前先尝试lock(), 离开临界区之前需要unlock()。
  
* std::lock_guard

  std::lock_guard提供了一个符合RAII的使用互斥量的手段。一般把lock_guard作为线程栈区的局部变量，通过初始化lock_guard可以对传入的mutex进行锁定，当lock_guard对象被析构，那么它拥有的mutex也会被解锁。
  
  ```cpp
  std::mutex mymutex;
  void threadrun()
  {
  	std::lock_guard<std::mutex> lock(mymutex);
  	
  	//临界区
  	
  }  
  ```
  
  可以发现，lock_guard在使用方面更加方便，降低犯错的可能性。
  
* std::unique_lock
  
  这是C++11提供的最灵活的锁，可以灵活的控制所拥有的锁的状态，甚至可以在栈之间进行锁的转移。但是呢，性能和lock_guard相比，有所下降。如果可以用lock_guard解决的业务场景，可以不用unique_lock。unique_lock提供std::adopt_lock和std::defer_lock第二参数，可以提供超强的灵活性。同时，unique_lock是典型的*不可复制*但是*可移动*对象。
  
  ```cpp
  
  std::mutex mymutex;
  void threadrun()
  {
  	mymutex.lock();
  	std::unique_lock<std::mutex> lock(mymutex,std::adopt_lock);
  	//临界区
  
  }  
  ```
  
  以上代码是在mutex已经被锁定的情况转移控制权到unique_lock对象。
  
  ```cpp
  std::mutex mymutex;
  void threadrun()
  {
  	std::unique_lock<std::mutex> lock(mymutex,std::defer_lock);
  	lock.lock();
  	//临界区
  
  }
  ```
  
  以上代码是unique_lock对象初始化的时候不进行锁定，等到实际需要的时候可以再锁定。
  
互斥量虽然提供了很方便的临界区管理工具，但是，滥用互斥量也会导致*死锁*。解除死锁，除了破除计算机考研的死锁的四个条件外，其实，避免死锁有一个最简单的准则，就是每个线程获取互斥量的顺序必须绝对一致。

C++11提供了一个非常好的工具来避免线程获取多个锁的时候死锁，std::lock可以同时锁定多个锁，同时还不会发生死锁。

```cpp
std::mutex m1;
std::mutex m2;

void threadrun()
{
  std::lock(m1,m2);
  std::unique_lock<std::mutex> lock1(m1,std::adopt_lock);
  std::unique_lock<std::mutex> lock2(m2,std::adopt_lock);
}

```

除了以上方式，还可以采用std::defer_lock。

```cpp
std::mutex m1;
std::mutex m2;

void threadrun()
{
  std::unique_lock<std::mutex> lock1(m1,std::defer_lock);
  std::unique_lock<std::mutex> lock2(m2,std::defer_lock);
  std::lock(lock1,lock2);
}

```

  

  
  
