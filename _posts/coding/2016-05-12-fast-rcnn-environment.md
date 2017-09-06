---
layout: post
title:  "Fast_rcnn+Caffe配置以及部分问题汇总"
category: coding
tags: [caffe,Fast-rcnn]
description: caffe
---  

### 序

搭建Fast_rcnn是在搭成功Caffe的基础上进行的，请看之前的文章系列Caffe配置 
前面的Caffe系列用了Anaconda,事实证明虽然Caffe搭建成功了，但是fast-rcnn错误百出，于是又切换到系统python形式，重新编译的Caffe。

### fast-rnn配置

1 克隆安装包  
```
git clone --recursive https://github.com/rbgirshick/fast-rcnn.git   
```
2 安装python依赖包
```
sudo apt-get install python-pip
sudo pip install cython
sudo apt-get install python-opencv
sudo pip install easydict
```

生成Cpython模块
```
cd fast-rcnn/lib
sudo make
```

3 编译
```
cd caffe-fast-rcnn
cp caffe-master/Makefile.config
caffe-fast-rcnn/
sudo make && make pycaffe
```

4 下载测试模型

重新进入fast-rcnn文件夹下

```
cd fast-rcnn
./data/scripts/fetch_fast_rcnn_models.sh
```

我的模型就是半天没下载下来，在此给出资源： 
链接: [https://pan.baidu.com/s/1o7W0S6u](https://pan.baidu.com/s/1o7W0S6u) 密码: 2bty

5 python运行demo

进入fast-rcnn文件夹下，运行
```
sudo ./tools/demo.py --net caffenet
```
可能出现No module named skimage.io 
cd ~之后再输入
```
sudo pip install scikit-image
```
再install时，可能又会出现 
not find scipy 
此时输入：

```
sudo pip install scipy 
```

显然它还有错误：
```
error: library dfftpack has Fortran sources but no Fortran compiler found
```

请再次耐心输入：
```
sudo apt-get install gfortran
```
6 总结

前方路上艰难险阻，多次想要退出，但是还是感谢网上的伙伴给出的各种解决方案，更要感谢舍长的无私帮助，看到检测窗口出现的刹那，真的开心！开启了一片新世界！