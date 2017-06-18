---
layout: post
title:  "最短路径Dijitstra算法"
category: [algorithm ]
tags: [c++,algorithms]
description: 最短路径Dijitstra算法
header-img: "img/pages/template.jpg"
---

### 最短路径Dijitstra算法

#### 1、算法简介

![如图](http://images.cnitblog.com/blog/387014/201308/20161121-468604458fb64b7ca02f75e22c4d3d15.png)

#### 2、算法步骤
* 1.设定flag[length] 用于标定是否访问过该点
* 2.初始化所有点的距离为0，并且初始化flag为0
* 3.将节点本身的dist设为0
* 4.找到未访问点的最小距离
* 5.用该点对前面所有点进行更新

#### 3、代码如下（想法清晰）

```c++
void Dijistra(vector<vector<int > >map ,vector<int> &dist , int start)
{
    int length= map.size();
    vector<int> flag(length,-1);
    for(int i = 0; i < length;i++)
    {
        if( start == i)
           dist[i] = 0;
        else
            dist[i] = map[start][i];  //初始化所有点
    }
    while(true)
    {
           int min = INT_MAX;
           int index = -1;
           for(int i = 0; i< length;i++)
           {
              if(flag[i] != -1)
              {
                 if(index==-1 || dist[index] > dist[i])
                    index = i;
              }
           }
           if(index == -1)
              return;
           flag[index] = true;
           for(int i = 0; i< length;i++)
           {
               if(map[index][i]!= MAX)
               {
                  if(dist[i]==-1 || dist[i] > dist[index] + map[index][i])
                     dist[i] = dist[index] + map[index][i];
                }
           }
    }
    
}
```

#### 4、可运行代码

```c++
void dijkstra(const int &beg,//出发点
	const vector<vector<int> > &adjmap,//邻接矩阵，通过传引用避免拷贝
	vector<int> &dist,//出发点到各点的最短路径长度
	vector<int> &path)//路径上到达该点的前一个点
	//负边被认作不联通
	//福利：这个函数没有用任何全局量，可以直接复制！
{
	const int &NODE = adjmap.size();//用邻接矩阵的大小传递顶点个数，减少参数传递
	dist.assign(NODE, -1);//初始化距离为未知
	path.assign(NODE, -1);//初始化路径为未知
	vector<bool> flag(NODE, 0);//标志数组，判断是否处理过
	dist[beg] = 0;//出发点到自身路径长度为0
	while (1)
	{
		int v = -1;//初始化为未知
		for (int i = 0; i != NODE; ++i)
		if (!flag[i] && dist[i] >= 0)//寻找未被处理过且
		if (v<0 || dist[i]<dist[v])//距离最小的点
			v = i;
		if (v<0)return;//所有联通的点都被处理过
		flag[v] = 1;//标记
		for (int i = 0; i != NODE; ++i)
		if (adjmap[v][i] >= 0)//有联通路径且
		if (dist[i]<0 || dist[v] + adjmap[v][i]<dist[i])//不满足三角不等式
		{
			dist[i] = dist[v] + adjmap[v][i];//更新
			path[i] = v;//记录路径
		}
	}
}

```





