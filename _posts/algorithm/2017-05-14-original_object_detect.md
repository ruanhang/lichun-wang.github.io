---
layout: post
title:  "传统物体检测"
category: [algorithm ]
tags: [object detect,algorithms]
description: Sort
header-img: "img/pages/template.jpg"
--


1.1 图像数据库汇总



![图像数据库](http://img.blog.csdn.net/20170906200237168?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzEyOTQyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
附数据集链接（知乎达人收集） 
[https://zhuanlan.zhihu.com/p/25138563](https://zhuanlan.zhihu.com/p/25138563)

1.2 物体检测

传统物体检测是滑动窗口设计+特征提取+分类器的方式。  

1) 滑动窗口设计：  

穷举策略：图像中所有可能出现框的位置从左往右，从上往下遍历一次，并缩放一组图像尺寸，得到图像金字塔进行多尺度搜索。存在问题：时间复杂度高，窗口冗余。  

2) 特征提取  

Harr角点检测：提取多种边缘变化信息，快速计算。   
LBP: 提取纹理信息，对均匀变化光照有很强适应性。   
HOG: 对物体边缘使用直方图统计进行编码，特征能力强。   
SIFT: 提取局部特征，具有尺度不变性。   
特征设计需要研究者经验驱动，组合不同特征，进行调优。  

3) 分类器设计  

Adaboost, SVM, DT，RF等。  


