---
title: 'KMP(看毛片)算法C++实现'
date: 2017-09-28 01:13:20
tags: Algorithm
---

看毛片算法也是堪称经典，理解起来略费事，不过搞清楚前缀、后缀其实也不困难，和传统字符匹配算法不同的是，KMP跳接的跨度更大，需要辅助next[]

下一跳跨度=上一次匹配的字符个数-next[上一次最后匹配字符的index]

<!--more-->

* KMP

```cpp
//
// Created by mayusheng on 2017/9/27.
//
#include <iostream>
#include <vector>
#ifndef ALGORITHM_KMP_HPP
#define ALGORITHM_KMP_HPP

void gennext(std::string & p, std::vector<u_int> &next){

    next.resize(p.length(),0);

    //生成next[]，最大共同前缀后缀长度

    for(int i=1;i<=p.length();i++)
    {
    std::string q = p.substr(0,i);

    int cusorA=1;
    int cusorB=q.length()-2;
    while(cusorA<q.length()&&cusorB>=0) {
        std::string pre = q.substr(cusorA, q.length() - cusorA);
        std::string las = q.substr(0, cusorB+1);
        if (pre == las)
        {
            if(pre.length()>next[i])
                next[i-1]=pre.length();
        }
        ++cusorA;
        --cusorB;
    }

    }



}

void KMP(std::string &text,std::string &p)
{
    std::vector<u_int> next;
    gennext(p,next);
    u_int cusor_text = 0;
    u_int cusor_p = 0;
    //看毛片(KMP)匹配，cusor_text需要调到cusor_text+(部分匹配长度-next[部分匹配长度-1])(由于数组为0开始）
    while(cusor_text<=text.length()-p.length())
    {

        cusor_p=0;
        while(text[cusor_text]==p[cusor_p]&&cusor_p<p.length())
        {
            cusor_p++;
            cusor_text++;
        }
        if(cusor_p==p.length())
            std::cout<<cusor_text-cusor_p<<"*"<<text.substr(cusor_text-cusor_p,p.length())<<std::endl;

        if(cusor_p!=0&&cusor_p!=p.length())
            cusor_text+=(cusor_p-next[cusor_p-1]);
        if(cusor_p==0)
            ++cusor_text;
        //cusor_p的调整需要用心考虑
    }




}


#endif //ALGORITHM_KMP_HPP
```
我的写法肯定效率不是很高，但是，可能是最适合现场撸的KMP版本，时候面试现场撸码。