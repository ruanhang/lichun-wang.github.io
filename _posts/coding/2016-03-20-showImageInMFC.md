---
layout: post
title:  "MFC��ʵ�� �Ҷ�ͼ����ʾ�������� C++"
category: coding
tags: [c++,MFC,ͼ����]
description: MFC��ʵ�� �Ҷ�ͼ����ʾ�������� C++
---  

---

### һ����������
 > ���ȣ�����ͼ���̵Ĺ����У�����ͼ�����ʾ�Ƿǳ���Ҫ�ģ����Ƕ���ͼ��Ĵ����������õ�ͼ�����ʾ�����ڲ鿴����Ƿ���ȷ������������Ȼ�����һ���뷨���Ƿ���Ա�дһ��ͼ����ʾ�ĺ�����������matlab����openCV�е�imshow()������������ʱ��ͼ���ӡ�����أ��𰸱�Ȼ�ǿ϶��ģ������������mfc������ʵ���Լ���imshow()������������mfc�����£�vc++Ϊ�����ṩ�˺ܶ��ݵĽṹ��ͺ�����ʹ�������ܷ��㡣

---

### �����������
> ��ʾͼ��������Ҫ�õ��ĺ�����**SetDIBitsToDevice()**;���������������������ñ��ǽ�ͼ����ʾ�������ʾ�����棬��������ϸ�����������ұ㲻����ϸ�����ˣ����߿������еĲ鿴�ٶȰٿ����ӣ�[http://baike.baidu.com/linkurl=D8WZ6hoanRGCSCrjCia5BKJli5saxSmdi8guNtlvrrlbUxy1BF52o5q1LwImvvHQ5gRCxZMfS0HdrF0C0kv4Q_](https://www.baidu.com/swd=SetDIBitsToDevice&rsv_spt=1&rsv_iqid=0x9b522a5b00009696&issp=1&f=8&rsv_bp=0&rsv_idx=2&ie=utf-8&tn=baiduhome_pg&rsv_enter=1&rsv_sug3=2&rsv_sug1=1&rsv_n=2)��

### ������Ҫ�ӿں�������
> ������Ҫ����һ�º�����
   ����ԭ��Ϊ��
  int SetDIBitsToDevice(HDC hdc, int xDest, int Ydest, DWORD dwWidth, DWORD dwHeight, intXSrc, int Ysrc, UINT uStartScan, UINT cScanLines, CONST VOID *lpvBits, CONST BITMAPINFO *lpbmi, UINT fuColorUse)��

 ~~~ c++
/*�����������£�*/
hdc��/*�豸�������*/
XDest, YDest ��/*��ʾ����Ļ��������Ͻǵ�����ꡣ*/
dwWidth��/*ͼ���ȡ�*/
dwHeight��/*ͼ��߶ȡ�*/
XSrc ��YSrc �� /*ͼ�����ʼ���� ��һ��˴�Ϊ 0 0 ��*/   /*��ʵ, XSrc ,YSrc dwWidth dwHeight���ǽ�ͼ��ľ���ȷ������*/
uStartScan�� /*ָ��DIB�е���ʼ[ɨ����]���˴�һ��Ϊ 0 ��*/
cScanLInes�� /*ָ������lpvBitsָ��������а�����DIBɨ������Ŀ��*/  /*˵���˾���ͼ��ĸ߶�*/
lpvBits�� /*ָ��洢DIB��ɫ���ݵ��ֽ�����[����]��ָ�롣*/
lpbmi�� /*ָ��BITMAPINFO�ṹ��ָ�룬�ýṹ�����й�DIB����Ϣ��*/
/*fuColorUse�� ָ��BITMAPINFO�ṹ�еĳ�ԱbmiColors�Ƿ������ȷ��RGBֵ��Ե�ɫ�����������ֵ���йظ������Ϣ����ο�����ı�ע���֡�*/
/*����fuColorUse����������ֵ֮һ����Щֵ�ĺ������£�*/
    /*1��DIB_PAL_COLORS����ʾ��ɫ����16λ������ֵ[����]��ɣ�������Щֵ�ɶԵ�ǰѡ�е��߼���ɫ�����������*/
   /* 2��DIB_RGB_COLORS����ʾ��ɫ�����ԭ���RGBֵ��*/
  /*�ر�ע�⣺����Ĭ�ϵ���ʾ����ϵ����������������ϵ���෴�ģ�������������ô˺���������ʾ������ͼ���ǵ����ģ���ε���������*/
   /*��������������֮�����Ǳ���Խ��б���ˡ�*/
~~~

### �ġ�ʵ�ִ���
>����˵�� ��
       ��ʾ����ΪshowGrayImg(),���������ֱ�Ϊ��ͼ��ָ�룬ͼ���ȣ��Լ�ͼ��߶ȡ�
   ��ʾͼ���У����ȵ��� setBitMapInfo()��������ͼ����ļ�ͷ������setDIBitsToDevice()�����ĵ��á�
     **�ر�ע�⣬����Ķ���ͼ����Ϊ4���������ĵ���������ͼ��߽���룬��ͼ��ÿһ�еĴ洢��С��Ϊ4����������**���ˣ��˺�������������ڳ�������ʱ��ʾ�����ˣ���ͬ��matlab�е�imshow()������

~~~ c++
void setBitmapInfo(BITMAPINFO *bitmapInfo,int width,int height)
{   
         bitmapInfo->bmiHeader.biSize  = sizeof(BITMAPINFOHEADER);
         bitmapInfo->bmiHeader.biWidth    = width;   
         bitmapInfo->bmiHeader.biHeight   = -height;    //�ر�ע��˴�Ҫ��Ϊ��ֵ�����ڵ���ͼ�����ʾ�������Ϊ+����ͼƬ����ʾΪ������ͼ��Ϊ����Ϊ������ͼ��     bitmapInfo->bmiHeader.biPlanes   = 1;
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
     BITMAPINFO * bitmapInfo = (BITMAPINFO*)new    BYTE[sizeof(BITMAPINFOHEADER)+256*sizeof(RGBQUAD)];  //����bmpͷ�Լ���ɫ��ռ�   
     setBitmapInfo(bitmapInfo,width,height);   //����bmpͷ�ļ��Լ���ɫ��   
     int newWidth = (width + 3)/4*4;  //���ڵ���ͼ����Ϊ4������������Ϊ��ʾ��ʱ��Ҫ����ǰ��մ洢�е���ʽ������ʾ��   int count = 0;   
     BYTE * memImg = new BYTE[newWidth * height];  //���ٿռ����ھ�ͼ���ȶ���  
     memset(memImg,0,newWidth*height);  
     for(;count<height;count++)  
     {     
        memcpy (memImg+count*newWidth,img+count*width,width);  //������ʾ�߽�  
     }   
     HDC hDC= GetDC( GetForegroundWindow() );          //��ȡ��ǰ��ʾ���ľ��   
     SetDIBitsToDevice(hDC,100,70,width,height,0,0,   0,height,memImg,bitmapInfo,DIB_RGB_COLORS);  
     delete []bitmapInfo;   
     delete []memImg;   
     return 0;
 }
~~~

