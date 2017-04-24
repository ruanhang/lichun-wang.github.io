---
layout: post
title:  "MFC下实现 灰度图像显示函数代码 C++"
category: coding
<<<<<<< HEAD
tags: [c++,MFC]
description: MFC,C++
=======
tags: [c++,MFC,图像处理]
description: MFC下实现 灰度图像显示函数代码 C++
>>>>>>> origin/master
---  

---

### 函数原型
> 函数原型
  int SetDIBitsToDevice(HDC hdc, int xDest, int Ydest, DWORD dwWidth, DWORD dwHeight, intXSrc, int Ysrc, UINT uStartScan, UINT cScanLines, CONST VOID *lpvBits, CONST BITMAPINFO *lpbmi, UINT fuColorUse)��

 ~~~ c++
 hdc：设备环境句柄
XDest,  YDest  ：//显示在屏幕上面的左上角点的坐标。
dwWidth：//图像宽度。
dwHeight：//图像高度。
XSrc ，YSrc ： //图像的起始坐标 ，一般此处为 0  0  。      /*其实, XSrc ,YSrc dwWidth dwHeight便是将图像的矩形确定出来*/
uStartScan： //指定DIB中的起始扫描线，此处一般为 0 。
cScanLInes：// 指定参数lpvBits指向的数组中包含的DIB扫描线数目。   /*说白了就是图像的高度*/
lpvBits： //指向存储DIB颜色数据的字节类型数组的指针。
lpbmi： //指向BITMAPINFO结构的指针，该结构包含有关DIB的信息。
fuColorUse： //指向BITMAPINFO结构中的成员bmiColors是否包含明确的RGB值或对调色板进行索引的值。有关更多的信息，请参考下面的备注部分。
//参数fuColorUse必须是下列值之一，这些值的含义如下：
DIB_PAL_COLORS：//表示颜色表由16位的索引值数组组成，利用这些值可对当前选中的逻辑调色板进行索引。
DIB_RGB_COLORS：//表示颜色表包含原义的RGB值。
=======
### 一、问题描述
 > 首先，在做图像编程的过程中，对于图像的显示是非常重要的，我们对于图像的处理，经常会用到图像的显示，用于查看结果是否正确，所以我们自然会产生一个想法，是否可以编写一个图像显示的函数，类似于matlab或者openCV中的imshow()函数，可以随时将图像打印出来呢？答案必然是肯定的，在这里，我们在mfc环境下实现自己的imshow()函数。并且在mfc环境下，vc++为我们提供了很多便捷的结构体和函数，使用起来很方便。

---

### 二、解决方法
> 显示图像，我们主要用到的函数是**SetDIBitsToDevice()**;这个函数，这个函数的作用便是将图像显示在输出显示器上面，函数的详细介绍在这里我便不做详细阐述了，读者可以自行的查看百度百科链接（[http://baike.baidu.com/linkurl=D8WZ6hoanRGCSCrjCia5BKJli5saxSmdi8guNtlvrrlbUxy1BF52o5q1LwImvvHQ5gRCxZMfS0HdrF0C0kv4Q_](https://www.baidu.com/swd=SetDIBitsToDevice&rsv_spt=1&rsv_iqid=0x9b522a5b00009696&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=2&rsv_sug1=1&rsv_n=2)）

### 三、主要接口函数介绍
> 这里主要介绍一下函数。
   函数原型为：
  int SetDIBitsToDevice(HDC hdc, int xDest, int Ydest, DWORD dwWidth, DWORD dwHeight, intXSrc, int Ysrc, UINT uStartScan, UINT cScanLines, CONST VOID *lpvBits, CONST BITMAPINFO *lpbmi, UINT fuColorUse)；
~~~ c++
参数定义如下：
hdc：设备环境句柄
XDest, YDest ：显示在屏幕上面的左上角点的坐标。
dwWidth：图像宽度。
dwHeight：图像高度。
XSrc ，YSrc ： 图像的起始坐标 ，一般此处为 0 0 。   /*其实, XSrc ,YSrc dwWidth dwHeight便是将图像的矩形确定出来*/
uStartScan： 指定DIB中的起始[扫描线]，此处一般为 0 。
cScanLInes： 指定参数lpvBits指向的数组中包含的DIB扫描线数目。  /*说白了就是图像的高度*/
lpvBits： 指向存储DIB颜色数据的字节类型[数组]的指针。
lpbmi： 指向BITMAPINFO结构的指针，该结构包含有关DIB的信息。
fuColorUse： 指向BITMAPINFO结构中的成员bmiColors是否包含明确的RGB值或对调色板进行索引的值。有关更多的信息，请参考下面的备注部分。
参数fuColorUse必须是下列值之一，这些值的含义如下：
    1、DIB_PAL_COLORS：表示颜色表由16位的索引值[数组]组成，利用这些值可对当前选中的逻辑调色板进行索引。
    2、DIB_RGB_COLORS：表示颜色表包含原义的RGB值。
  /*特别注意：由于默认的显示坐标系与我们正常的坐标系是相反的，所以如果仅仅用此函数正常显示出来，图像是倒立的，如何调整见程序*/
   对于这个函数理解之后我们便可以进行编程了。
>>>>>>> origin/master
~~~

### 四、实现代码
>函数说明 ：
       显示函数为showGrayImg(),三个参数分别为，图像指针，图像宽度，以及图像高度。
   显示图像中，首先调用 setBitMapInfo()函数设置图像的文件头，用于setDIBitsToDevice()函数的调用。
     **特别注意，后面的对于图像宽度为4的整数倍的调整，并将图像边界对齐，即图像每一行的存储大小都为4的整数倍。**到此，此函数便可以用于在程序中随时显示函数了，等同于matlab中的imshow()函数。

~~~ c++
void setBitmapInfo(BITMAPINFO *bitmapInfo,int width,int height)
{   
         bitmapInfo->bmiHeader.biSize  = sizeof(BITMAPINFOHEADER);
         bitmapInfo->bmiHeader.biWidth    = width;   
         bitmapInfo->bmiHeader.biHeight   = -height;    //特别注意此处要设为负值，用于调整图像的显示方向，如果为+，则图片的显示为倒立的图像，为调整为正立的图像。     bitmapInfo->bmiHeader.biPlanes   = 1;
         bitmapInfo->bmiHeader.biBitCount  = 8;
         bitmapInfo->bmiHeader.biCompression = 0;   
         bitmapInfo->bmiHeader.biSizeImage  = (width+3)/4*4 * height;  
         bitmapInfo->bmiHeader.biXPelsPerMeter = 0;  
         bitmapInfo->bmiHeader.biYPelsPerMeter = 0;  
         bitmapInfo->bmiHeader.biClrUsed    = 0;   
         bitmapInfo->bmiHeader.biClrImportant = 0;  
         int count = 0;    
        for(count=0;count<256;count++) 
        {   
           bitmapInfo->bmiColors[count].rgbBlue = count;   
           bitmapInfo->bmiColors[count].rgbGreen = count; 
           bitmapInfo->bmiColors[count].rgbRed = count;    
           bitmapInfo->bmiColors[count].rgbReserved = 0;  
       }
 }
int showGrayImg(BYTE * img,int width,int height)
{    
     BITMAPINFO * bitmapInfo = (BITMAPINFO*)new    BYTE[sizeof(BITMAPINFOHEADER)+256*sizeof(RGBQUAD)];  //开辟bmp头以及调色板空间   
     setBitmapInfo(bitmapInfo,width,height);   //设置bmp头文件以及调色板   
     int newWidth = (width + 3)/4*4;  //用于调整图像宽度为4的整数倍，因为显示的时候要求的是按照存储中的形式进行显示。   int count = 0;   
     BYTE * memImg = new BYTE[newWidth * height];  //开辟空间用于经图像宽度对齐  
     memset(memImg,0,newWidth*height);  
     for(;count<height;count++)  
     {     
        memcpy (memImg+count*newWidth,img+count*width,width);  //调整显示边界  
     }   
     HDC hDC= GetDC( GetForegroundWindow() );          //获取当前显示器的句柄   
     SetDIBitsToDevice(hDC,100,70,width,height,0,0,   0,height,memImg,bitmapInfo,DIB_RGB_COLORS);  
     delete []bitmapInfo;   
     delete []memImg;   
     return 0;
 }
~~~

