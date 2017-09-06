---
layout: post
title:  "Python-Numpy小结1"
category: coding
tags: [python]
description: python
---  

### 1 基础  
#### 1.1 Numpy主要对象是多维数组，尺寸成为轴，如：  
```python
[1,2,1]#1个轴，长度为3
[[1,0,1],
[0,1,2]]#2个轴，第一维长度2，第二维长度3
```

其数组类为ndarray,其属性为：  

```
ndarray.ndim#轴数
ndarray.shape#尺寸，(n,m),n行m列
ndarray.size#数组的元素和，等于形状元素的乘积
ndarray.dtype#数组中元素类型的对象
ndarray.itemsize#数组的每个元素大小（单位：字节）
```

#### 1.2 numpy.ndarray初始化  
```
array([[1,2,3],[4,5,6]])
arange(10,30,5) #间隔为5
x = linspace( 0, 2*pi, 100 )#共100个元素， 用于浮点数
zeros((3,4)), ones((2,3,4)), empty((2,3))
```

#### 1.3 打印数组  

* last轴的元素从左到右
* second-last 元素从上到下
* the rest 元素从上到下

#### 1.4 基本操作  
```
A*B #对应元素相乘
dot(A,B) #点乘，内积
A.sum(axis=0)#sum of each column
A.min(axis=1)#min of each row
A.max()
```

#### 1.5 索引，切片，迭代（Indexing, Slicing and Iterating）    

一维 Indexing:  
```
a[2]
a[2:5]
a[:6:2]#索引位置0-6，间隔为2
a[::-1]#逆序数组
```
二维Indexing:   
每个轴有一个索引，逗号分隔。  
```
b=empty((4,5))
b[2,3]
b[0:5, 1]=b[ : ,1]# each row in the second
b[1:3, : ]#each column in the second
b[-1]=b[-1,:]#last row
```

迭代：   
默认按行输出  
```
b=arrar([[0 1 2 3],
[10 11 12 13]])
for row in b:print row
>>>
[0 1 2 3]
[10 11 12 13]
```

使用flat访问每个元素  

```
for element in b.flat:
         print element,
0 1 2 3 10 11 12 13
```
#### 1.6 Shape Manipulation  

改变形状  
``` 
a = floor(10*random.random((3,4)))#floor向下取整函数
a.ravel()#flatten the array
a.shape
a.transpose()#转置
a.resize(2,6)
a.resize(3,-1)#-1 为自动计算
```

叠加数组  
```
>>> a = floor(10*random.random((2,2)))
>>> a
array([[ 1.,  1.],
       [ 5.,  8.]])
>>> b = floor(10*random.random((2,2)))
>>> b
array([[ 3.,  3.],
       [ 6.,  0.]])
>>> vstack((a,b))
array([[ 1.,  1.],
       [ 5.,  8.],
       [ 3.,  3.],
       [ 6.,  0.]])
>>> hstack((a,b))
array([[ 1.,  1.,  3.,  3.],
       [ 5.,  8.,  6.,  0.]])
```
column_stack #按列叠加数组，与vstack不同;row_stack #按行叠加数组  
```python
>>> column_stack((a,b))   # With 2D arrays
array([[ 1.,  1.,  3.,  3.],
       [ 5.,  8.,  6.,  0.]])
>>> a=array([4.,2.])
>>> b=array([2.,8.])
>>> a[:,newaxis]  # This allows to have a 2D columns vector
array([[ 4.],
       [ 2.]])
>>> column_stack((a[:,newaxis],b[:,newaxis]))
array([[ 4.,  2.],
       [ 2.,  8.]])
>>> vstack((a[:,newaxis],b[:,newaxis])) # The behavior of vstack is different
array([[ 4.],
       [ 2.],
       [ 2.],
       [ 8.]])
```
留坑。。。分割数组，没看懂  
```python
>>> a = floor(10*random.random((2,12)))
>>> a
array([[ 8.,  8.,  3.,  9.,  0.,  4.,  3.,  0.,  0.,  6.,  4.,  4.],
       [ 0.,  3.,  2.,  9.,  6.,  0.,  4.,  5.,  7.,  5.,  1.,  4.]])
>>> hsplit(a,3)   # Split a into 3
[array([[ 8.,  8.,  3.,  9.],
       [ 0.,  3.,  2.,  9.]]), array([[ 0.,  4.,  3.,  0.],
       [ 6.,  0.,  4.,  5.]]), array([[ 0.,  6.,  4.,  4.],
       [ 7.,  5.,  1.,  4.]])]
>>> hsplit(a,(3,4))   # Split a after the third and the fourth column
[array([[ 8.,  8.,  3.],
       [ 0.,  3.,  2.]]), array([[ 9.],
       [ 9.]]), array([[ 0.,  4.,  3.,  0.,  0.,  6.,  4.,  4.],
       [ 6.,  0.,  4.,  5.,  7.,  5.,  1.,  4.]])]
```