---
layout: post
title:  "利用 OpenCV 在MFC中显示图像问题以及解决方法"
category: coding
tags: [c++,OpenCV,MFC]
description:
---
---
最近在做一个项目，需要在MFC中显示OpenCV读取的图像，遇到了一些问题，现在总结如下，希望对大家有帮助。
---
 ## 问题1：如何在MFC控件中显示OpenCV读取的图像
---
 ### 1.1问题说明
在做工程项目的时候遇到了这样一个问题，将用OpenCV读取的图像（Mat类型，或者IPlImage类型）显示在MFC的Picture控件中，那么将如何才能方便的显示呢？
 ### 1.2解决方法
经过研究发现如下两种方法：

* 1、利用CvvImage类，可以方便的在MFC对应控件中显示图像，方法如下：
首先，由于从OpenCV 2.2.0开始，OpenCV取消了CvvImage这个类，具体原因暂时不太清楚，所以导致OpenCV2.2后面的版本无法直接使用这个类，但是这个类对于MFC的显示确实非常的简单，所以为了继续使用这个类，我们可以下载CvvImage的源码，将CvvImage.cpp以及CvvImage.h添加到工程中去（注：CvvImage.cpp需要在开头加上预编译头文件 #include "stdafx.h" ）！[下载链接](http://download.csdn.net/detail/abc123abc_123/5721905)，添加到工程之后便可以利用CvvImage进行显示了。并且由于CopyOf后cimg空间不会自动回收，所以不要忘记手动释放内存。

~~~ C++
       Mat mat = imread(filePath);
       CDC* pDC = GetDlgItem( ID )->GetDC();
       HDC hDC = pDC->GetSafeHdc();
      IplImage img = mat;
      CvvImage cimg;
      cimg.CopyOf(&img);
      CRect rect;
      GetDlgItem( ID )->GetClientRect(&rect);
      cimg.DrawToHDC(hDC, &rect);
      cimg.Destroy();  //注意释放空间
      ReleaseDC(pDC);     //释放
~~~

* 2、利用c++以及windows系统函数进行显示,方法如下：
     
主要利用 StretchDIBits函数将图像数据显示到对应控件中，对于StretchDIBits具体含义，读者可以自行百度，这里给出显示函数代码以及主函数代码，注意在显示的时候，存在数据对其的问题，由于数据存储要求4字节对其，可能需要对显示的数据进行调整，int NewWidth = (width*(bit / 8) + 3) / 4 * 4，请读者注意。

主函数调用：

        IplImage *frame = cvLoadImage("path");
        DrawPicToHDC((BYTE *)frame->imageData, IDC_VIDEO, frame->width, frame->height, 24);
        cvReleaseImage(&frame);
    
显示函数：
~~~ C++
    void DrawPicToHDC(BYTE *img, UINT ID, int width, int height, int bit)
    {
    if (img == NULL)
        return;
    CWnd* pwd = AfxGetApp()->GetMainWnd()->GetDlgItem(ID);
    CDC *pDC = pwd->GetDC();
    CRect rect;
    pwd->GetClientRect(&rect);
    //pwd->RedrawWindow();
    BITMAPINFO* pInfo;
    if (bit == 24)
        pInfo = (BITMAPINFO*)new BYTE[sizeof(BITMAPINFO)];
    else if (bit == 8)
    {
        pInfo = (BITMAPINFO*)new BYTE[sizeof(BITMAPINFO) + 256 * sizeof(pInfo->bmiColors)];
        for (int i = 0; i<256; i++)
        {
            pInfo->bmiColors[i].rgbRed = i;
            pInfo->bmiColors[i].rgbGreen = i;
            pInfo->bmiColors[i].rgbBlue = i;
            pInfo->bmiColors[i].rgbReserved = 0;
        }
    }
    else
    {
        AfxMessageBox("显示视频遇到严重错误");
        exit(0);
    }
    pInfo->bmiHeader.biSize = sizeof(BITMAPINFOHEADER);
    pInfo->bmiHeader.biWidth = width;
    pInfo->bmiHeader.biHeight = -height;//图像数据倒着存储，所以-height显示为正，如果为+height图像倒立
    pInfo->bmiHeader.biPlanes = 1;
    pInfo->bmiHeader.biBitCount = bit;
    pInfo->bmiHeader.biCompression = BI_RGB;
    pInfo->bmiHeader.biSizeImage = 0;
    pInfo->bmiHeader.biXPelsPerMeter = 0;
    pInfo->bmiHeader.biYPelsPerMeter = 0;
    pInfo->bmiHeader.biClrUsed = 0;
    pInfo->bmiHeader.biClrImportant = 0;
    int NewWidth = (width*(bit / 8) + 3) / 4 * 4;
    BYTE* NewImg = new BYTE[NewWidth*height];
    memset(NewImg, 0, NewWidth*height);
    for (int i = 0; i<height; i++)
    {
        memcpy(NewImg + i*NewWidth, img + i*width*(bit / 8), width*(bit / 8));
    }
    SetStretchBltMode(pDC->GetSafeHdc(), COLORONCOLOR);
    StretchDIBits(pDC->GetSafeHdc(), 0, 0, rect.Width(), rect.Height(), 0, 0, width, height, NewImg, pInfo, DIB_RGB_COLORS, SRCCOPY);
    delete[] pInfo;
    delete[] NewImg;
    pwd->ReleaseDC(pDC);
    }
~~~

截止目前，图像的显示已经基本实现了，但是到此运行之后你会发现一个奇妙的现象，内存泄漏。

<center>
    <p><img src="http://upload-images.jianshu.io/upload_images/2829844-f3386e5c30e079ad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" align="center"></p>
</center>

---

## 问题2：出现内存泄漏
###2.1问题说明
>在显示实现的时候，明明没有自己new变量，为什么会出现内存泄漏呢？并且在进行调试后发现，在MFC工程中，只要定义了Mat，或者IPlImage变量，即使运行结束释放掉，仍然会出现内存泄漏，这是什么原因呢？

###2.2解决方法
>经过查询发现出现内存泄漏的原因不是你代码有问题，而是MFC编译的问题，网上说，由于引入OpenCV库之后，OpenCV的链接库core.dll先与MFC的库文件生成，所以导致内存泄漏，解决方法是将MFC的动态编译该为静态编译，进行如下操作，在工程环境中依次选择：

>** 工程-- 属性--  配置属性 --常规 --MFC的使用 选择静态库使用 **

>到此，内存泄漏已经解决，但同时很可能出现另一个问题，程序崩溃。错误提示为 _pFirstBlock==pHead，如下图，此问题如下解决。

<center>
    <p><img src="(http://upload-images.jianshu.io/upload_images/2829844-b8e7facce8ff3353.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" align="center"></p>
</center>

---

## 问题3：解决_pFirstBlock==pHead导致程序崩溃
###3.1问题说明
>在问题2中内存泄漏的问题已经解决了，但是在进行开发的过程中，可能会发现程序崩溃的问题，特别是当程序比较庞大的时候更容易出现此问题，错误提示为：_pFirstBlock==pHead。

###3.2解决方法
>此问题的产生多半是因为在调用库的过程中产生了冲突，所以解决此问题的方法就是将OpenCV的调用方法改为静态调用， **使用OpenCV的静态库**
### opencv中在静态库中使用MFC的配置方法如下：

>* 1、lib选择staticlib；
也就是VC++目录中的包含目录应该为如下路径
D:/Program Files/opencv/build/x86/vc12/staticlib
* 2、属性页---配置属性----MFC的使用---在静态库下使用MFC；
这样会将你程序用到的一些库写到你的exe文件中，换来的是可移植性，但是exe文件会稍微大一些
* 3、属性----C/C++ -----代码生成----运行库选择位多线程调试(/MTd)。

在静态库下也可能会出现异常错误：
这时候考虑的问题有如下2个：

>* 1、确实是你程序错误，如果程序错误最有可能是你new的指针没有delete，或者某个内存没有分配就开始用再或者就是野指针等情况，最好单步调试，注意指针和数组。
* 2、opencv的配置错误
配置好opencv后发现我的程序在共享DLL下使用MFC是没有错误，但是一旦选择了静态库下使用MFC就出现了上面的错误。如果不是程序问题，那么通常，Debug下面可能引用了Release下面静态编译的库。如果在debug环境下运行，只要将release下面的库全部删除就可以了。
