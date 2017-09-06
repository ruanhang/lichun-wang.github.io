---
layout: post
title:  " spyder--no module named caffe"
category: coding
tags: [python,caffe]
description: python
---  

安装好了caffe, 在命令行窗口python解释器中可以正常import caffe，但是在spyder编译器里找不到caffe。
原因：系统路径不同，可以分别输入sys.path查看。
解决办法：
打开 Python-Path-Manager 路径管理器， 右边的python形状的图标，添加路径如图所示：

![1](http://img.blog.csdn.net/20170519103620172?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzEyOTQyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![2](http://img.blog.csdn.net/20170519103837772?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzEyOTQyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

重新启动spyder即可


