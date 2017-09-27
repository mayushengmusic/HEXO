---
title: 'Dijkstra算法堆优化C++实现'
date: 2017-09-28 01:13:20
tags: Algorithm
---

最近写了一些常见算法的C++实现，为以后的笔试或者比赛啥的做个模板储备，先来最短路径算法。

* Dijkstra

```cpp
//
// Created by mayusheng on 2017/9/27.
//
#include <list>
#include <queue>

#ifndef ALGORITHM_DIJKSTRA_HPP
#define ALGORITHM_DIJKSTRA_HPP

static std::vector<int> LittleHead;

void insertHead(int x,std::vector<int>& dis){//向最大堆插入元素
    if(LittleHead.empty()) {
        LittleHead.push_back(x);
        return;
    }

    LittleHead.push_back(x);
    for(int i=LittleHead.size()/2-2;i>=0;--i)
    {
        if(dis[2*i+1]==std::max(dis[LittleHead[2*i+1]],dis[LittleHead[2*i+2]]))
                std::swap(LittleHead[2*i+1],LittleHead[i]);
        else
                std::swap(LittleHead[2*i+2],LittleHead[i]);
    }



}

int  popHead(std::vector<int> & dis){//最大堆POP
    if(LittleHead.empty())
        return -1;

    int back = LittleHead.front();
    std::swap(LittleHead[0],LittleHead[LittleHead.size()-1]);

    LittleHead.erase(std::next(LittleHead.end(),-1));
    for(int i=LittleHead.size()/2-2;i>=0;--i)
    {
        if(dis[2*i+1]==std::max(dis[LittleHead[2*i+1]],dis[LittleHead[2*i+2]]))
            std::swap(LittleHead[2*i+1],LittleHead[i]);
        else
            std::swap(LittleHead[2*i+2],LittleHead[i]);
    }
    return back;

}

std::list<int> Dijkstra(std::vector<std::vector<int>> &mat,int start,int end){


    std::vector<int> Dis(mat.size(),std::numeric_limits<int>::max());
    std::vector<bool> isVisit(mat.size(), false);
    std::vector<int> lastCross(mat.size(),0);

    insertHead(start,Dis);//插入最大堆


    while(true){

        int node = popHead(Dis);//每次取的都是堆里面距离最小的
        if(node == -1||node == end)
            break;
        isVisit[node] = true;

        for(int i=0;i<mat[node].size();i++)
        {
            if(mat[node][i]!=std::numeric_limits<int>::max())
            {
                Dis[i]=std::min(Dis[i],Dis[node]+mat[node][i]);
                if(Dis[i]==Dis[node]+mat[node][i])
                    lastCross[i]=node;//记录最后一次更新Dis依赖的节点，可以得到路线
            }

            if(isVisit[i]==false)
                insertHead(i,Dis);

        }



    }

    std::list<int> ans;
    ans.push_back(end);
    while(end != start)
    {
        ans.push_front(end = lastCross[end]);
    }

    return ans;


}


#endif //ALGORITHM_DIJKSTRA_HPP
```
该算法主要加了一个最大堆辅助，可以提高选点效率。