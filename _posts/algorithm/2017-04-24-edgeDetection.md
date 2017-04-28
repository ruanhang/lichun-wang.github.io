---
layout: post
title:  "边缘检测总结"
category: [algorithm ]
tags: [c++,edge detection,algorithms]
description: 边缘检测
header-img: "img/pages/template.jpg"
---

### 一、Canny边缘检测
##### 1、原理摘要
	1、进行高斯滤波，去除椒盐噪声
	2、利用sobel边缘算子进行边缘检测并计算梯度以及方向（0,45,90,135度）
	3、局部梯度抑制，在梯度方向上，比较相邻是否为极大值
	4、双阈值分割，最大边缘少，最小连接在最大上面。

#### 2、OpenCV函数
	2.1函数原型
	Canny( InputArray _src, OutputArray _dst,  double low_thresh, double high_thresh,int aperture_size, bool L2gradient )
	函数输入：灰度图像，或者彩色图像
	函数输出：二值边缘图像
	函数功能：采用Canny方法对图像进行边缘检测
	
	2.2函数说明：
	1、第一个参数表示输入图像，必须为单通道灰度图。
	2、第二个参数表示输出的边缘图像，为单通道黑白图。
	3、第三个参数和第四个参数表示阈值，这二个阈值中当中的小阈值用来控制边缘连接，大的阈值用4、来控制强边缘的初始分割即如果一个像素的梯度大与上限值，则被认为是边缘像素，如果小于下限阈值，则被抛弃。如果该点的梯度在两者之间则当这个点与高于上限值的像素点连接时我们才保留，否则删除。
	5、第五个参数表示Sobel 算子大小，默认为3即表示一个3*3的矩阵。Sobel 算子与高斯拉普拉斯算子都是常用的边缘算子，详细的数学原理可以查阅专业书籍。


  



