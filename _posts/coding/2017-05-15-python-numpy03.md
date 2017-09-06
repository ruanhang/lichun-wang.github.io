---
layout: post
title:  "Python-Numpy小结3"
category: coding
tags: [python]
description: python
---  

### 1 布尔型索引，ix_()函数，矩阵和numpy数组

布尔索引

注意区分False和0，True和1 ，在这里这些是不同的。
```
a = arange(12).reshape(3,4)
b = a > 4
>>> a[b]
arrar([5,6,7,8,9,10,11])#false 不代表0 索引，而是直接不取出数据
```
ix_()函数

没完全搞懂什么意思，留坑，大致是组合向量，方便计算。
```
>>> a = array([2,3,4,5])
>>> b = array([8,5,4])
>>> c = array([5,4,6,8,3])
>>> ax,bx,cx = ix_(a,b,c)
>>> ax.shape, bx.shape, cx.shape
((4, 1, 1), (1, 3, 1), (1, 1, 5))
>>> result = ax+bx*cx
>>>result.shape
>>>(4,3,5)
```

matrices and 2D Arrays

首先看二者初始化的区别：
```
A = arange(3)
M = mat(A.copy())
##注意二者维度不同一个二维，一个一维，A.shape : (3,)  M.shape : (1,3)
```
取值时： M[0][n] A[n]

这就导致二者在索引时的不同，数组可以以逗号分隔，从而沿着多个轴进行索引
```
>>> print A[:,1]; print A[:,1].shape
[1 5 9]
(3,)
>>> print M[:,1]; print M[:,1].shape
[[1]
 [5]
 [9]]
(3, 1)
```

2D Arrays产生一维数组，矩阵产生二维数组，矩阵切片始终产生矩阵，数组切片则产生最低可能维数的数组。