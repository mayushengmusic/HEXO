---
title: 'linux系统静态库动态库探索'
date: 2016-12-30 22:57:00
tags: Linux
---

很长一段时间，对linux下软件的编译，运行的一些较为深入的细节很困惑，每次在linux下编译软件，都是./configure，make, make install三部曲。伴随着输出一大堆的古怪的文本。最近研究谷歌的protobuf，看了一些资料，总算对这个过程有所了解。我们借助protobuf生成一个C++类。有关protobuf的信息请移步[Protobuf](https://developers.google.com/protocol-buffers/).

![Linux Tux](https://i1.piimg.com/567571/c034afc0be5d5545.jpg)

<!--more-->
当然，写这篇文章最主要的还是加深自己的印象，同时方便日后查阅！我也是学习的网络上前人的文章，但是，可能有些小问题，通过自己实践，写了这篇文章。
首先说说静态库，所谓静态库，我的理解就是在编译过程中调用，但是，运行的时候并不需要在借助静态库，库里面可以糅合很多.o文件。动态库呢，就是运行的时候调用。
首先，我有一个类文件，person.pb.cc person.pb.h。不管是动态库还是静态库都需要.o文件，但是生成的时候会有区别。

* 动态库
如果是打算要生成静态库

```shg++ -c person.pb.cc```
然后你就能看到生成了person.pb.o,然后借助这个.o文件就可以生成静态库。

```shar cr libperson.a person.pb.o
```
OK.到此为止，我们就可以用这个静态库，这个库里面是person.pb.h里面声明的类的实现。
我的main.cpp调用了这个类里面的内容。

```shg++ -o run main.cpp -I./ -L./ -lprotobuf -lperson通过以上命令，就可以编译了。
```
*-I* 告诉编译器把当前目录下头文件包含到编译过程中。
*-L* 告诉编译器在当前目录下寻找库文件。
*-lprotobuf* 告诉编译器要链接libprotobuf.so,编译器会自动到/usr/lib找这个文件。
*-lperson* 告诉编译器去链接libperson.so,如果找不到动态库，才找静态库libperson.a。
到此为止，我们的静态库就链接成功，然后运行run，这里运行run，就不在需要.a静态库的文件了。

* 动态库
接下来我们尝试生成动态库
第一步还是要生成.o文件，切记，为了防止失败，不要用之前生成静态库的.o文件。

```shg++ -c -fpic person.pb.cc
```
接下来又会看到出现了一个新的person.pb.o文件，在执行以上命令之前，请删除原来的.o文件
好了，我们可以通过这个文件去生成.so的动态库了。

```shg++ -shared -fpic -o libperson.so person.pb.o
```
好了，我们还可以借助上面的main.cpp编译方法来生成我们的动态库链接版本。

```shg++ -o run main.cpp -I./ -L./ -lprotobuf -lperson
```
大功告成。试着运行一下，你会发现报错：

```sh./run: error while loading shared libraries: libperson.so: cannot open shared object file: No such file or directory
```

```shcp libperson.so /usr/lib
```
拷贝动态库到/usr/lib下，再次运行run就可以了。系统在运行run时，会自动到/usr/lib里面调用动态库，如果缺少动态库，当然会报错。
对以上了解以后，才能明白./configure存在的必要，还有make、cmake这些软件存在的必要性。
在Ubuntu系统下，一些lib软件包有-dev版本，我个人感觉，-dev版本应该就是把对应的头文件添加给系统，同时静态类库也安装到系统中。我们可以借助它来编译。普通版本，应该只有动态库吧。