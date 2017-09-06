---
layout: post
title:  "Python-Numpy小结2"
category: coding
tags: [python]
description: python
---  

[参考链接：http://scipy.github.io/old-wiki/pages/Tentative_NumPy_Tutorial](http://scipy.github.io/old-wiki/pages/Tentative_NumPy_Tutorial)

### 2 复制和视图  

简单操作不复制，函数引用不复制    
```
a = arange(12)  
b=a #a，b同一数组两个名字   
```

View和浅拷贝，创建新的数组对象，但是共享数据。特别的，列表切片返回View 。  
```
>>> c = a.view()
>>> c is a
False
>>> c.base is a                        # c is a view of the data owned by a
True
>>> c.flags.owndata
False
>>>
>>> c.shape = 2,6                      # a's shape doesn't change
>>> a.shape
(3, 4)
>>> c[0,4] = 1234                      # a's data changes
>>> a
array([[   0,    1,    2,    3],
       [1234,    5,    6,    7],
       [   8,    9,   10,   11]])
```

深拷贝   
完全拷贝数组和数据  
```
>>> d = a.copy()                          # a new array object with new data is created
>>> d is a
False
>>> d.base is a                           # d doesn't share anything with a
False
>>> d[0,0] = 9999
>>> a
array([[   0,   10,   10,    3],
       [1234,   10,   10,    7],
       [   8,   10,   10,   11]])
```

### 3 索引和技巧  

注：这部分知识以前未接触过，还是很新鲜的。    
```
>>> a = arange(12)**2                          # the first 12 square numbers
>>> i = array( [ 1,1,3,8,5 ] )                 # an array of indices
>>> a[i]                                       # the elements of a at the positions i
array([ 1,  1,  9, 64, 25])
>>> j = array( [ [ 3, 4], [ 9, 7 ] ] )         # a bidimensional array of indices
>>> a[j]                                       # the same shape as j
array([[ 9, 16],
       [81, 49]])
```

a为多维数组：  
```
>>> palette = array( [ [0,0,0],                # black
...                    [255,0,0],              # red
...                    [0,255,0],              # green
...                    [0,0,255],              # blue
...                    [255,255,255] ] )       # white
>>> image = array( [ [ 0, 1, 2, 0 ],           # each value corresponds to a color in the palette
...                  [ 0, 3, 4, 0 ]  ] )
>>> palette[image]                            # the (2,4,3) color image
array([[[  0,   0,   0],
        [255,   0,   0],
        [  0, 255,   0],
        [  0,   0,   0]],
       [[  0,   0,   0],
        [  0,   0, 255],
        [255, 255, 255],
        [  0,   0,   0]]])
```

索引为多个数组，必须维数相同：  
```
>>> a = arange(12).reshape(3,4)
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11]])
>>> i = array( [ [0,1],                        # indices for the first dim of a
...              [1,2] ] )
>>> j = array( [ [2,1],                        # indices for the second dim
...              [3,3] ] )
>>>
>>> a[i,j]                                     # i and j must have equal shape
array([[ 2,  5],
       [ 7, 11]])
>>>
>>> a[i,2]
array([[ 2,  6],
       [ 6, 10]])
>>>
>>> a[:,j]                                     # i.e., a[ : , j]
array([[[ 2,  1],
        [ 3,  3]],
       [[ 6,  5],
        [ 7,  7]],
       [[10,  9],
        [11, 11]]])
```

索引组可以以列表序列形式存储，但是不能存储为数组  
```
>>> l = [i,j]
>>> a[l]                                       # equivalent to a[i,j]
>>> s = array( [i,j] )
>>> a[s]                                       # not what we want
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
IndexError: index (3) out of range (0<=index<=2) in dimension 0
>>>
>>> a[tuple(s)]                                # same as a[i,j]
array([[ 2,  5],
       [ 7, 11]])
```

argmax() 返回最大值索引   
all() 全部   
any() 任意  

```
>>> time = linspace(20, 145, 5)                 # time scale
>>> data = sin(arange(20)).reshape(5,4)         # 4 time-dependent series
>>> time
array([  20.  ,   51.25,   82.5 ,  113.75,  145.  ])
>>> data
array([[ 0.        ,  0.84147098,  0.90929743,  0.14112001],
       [-0.7568025 , -0.95892427, -0.2794155 ,  0.6569866 ],
       [ 0.98935825,  0.41211849, -0.54402111, -0.99999021],
       [-0.53657292,  0.42016704,  0.99060736,  0.65028784],
       [-0.28790332, -0.96139749, -0.75098725,  0.14987721]])
>>>
>>> ind = data.argmax(axis=0)                   # index of the maxima for each series
>>> ind
array([2, 0, 3, 1])
>>>
>>> time_max = time[ ind]                       # times corresponding to the maxima
>>>
>>> data_max = data[ind, xrange(data.shape[1])] # => data[ind[0],0], data[ind[1],1]...
>>>
>>> time_max
array([  82.5 ,   20.  ,  113.75,   51.25])
>>> data_max
array([ 0.98935825,  0.84147098,  0.99060736,  0.6569866 ])
>>>
>>> all(data_max == data.max(axis=0))
True
```