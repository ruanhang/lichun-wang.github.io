---
layout: post
title:  "Opencv访问图像像素"
category: coding
tags: [ｃ++]
description: c++
---  

1.图像存储方式

图像在内存中的存储方式是连续的，每列连续存储RGB三个通道，可以用isContinuous()函数判断是否连续存储。下图为灰度图和彩色图的存储结构。 


![1](http://images2015.cnblogs.com/blog/993039/201608/993039-20160821150227058-1228580534.png)

![2](http://images2015.cnblogs.com/blog/993039/201608/993039-20160821150252558-1632770421.png)

2.访问图像像素方式

2.1 Mat 的at方法

at< Vec3b >(i,j)[k]的方式访问，其中i,j表示图像的行列，k表示通道数，RGB图像有三个通道，分别是0,1,2。如不需要访问图像每个通道，可以直接访问图像像素，则可用: at< uchar >(i,j) 方式直接去除图像特定行列的像素值。   
注：图像的像素点的灰度值类型必须为 uchar (unsigned char)，因为char: -128~127; uchar: 0~255。如果定义为char类型，在不需要对图像像素进行运算的时候是没问题的，但是一旦进行像素运算就有可能出错，为了避免出错，要习惯使用 uchar类型。 
特别的：OpenCV定义了一个Mat的模板子类为Mat_，它重载了operator()让我们可以更方便的取图像上的点。  
```
Mat_<uchar> im=outputImage;
im(i,j)=im(i,j)/div*div+div/2;
```

2.2指针法

Mat 的 uchar*p = ptr(i)的方式可以取出图像的行指针，这样访问图像数据的时候就可以遍历每行使用p[j]取出图像特定行列的像素值。 
该方法速度最快，效率最高，但是有可能出现越界问题。  

2.3迭代法  

采用 Mat_< Vec3b >::iterator it; 的方式声明一个迭代器，该方法比较安全，不会越界，但是效率低。  

上述方法的代码例子如下：
```
/*
主函数，声明了一个void colorReduce()函数，该函数分别用三种方式访问图像像素，运行例子的时候将不用的函数注释掉即可。
颜色空间缩减
对于三通道图像，一个像素对应的颜色有一千六百多万种，用如此多的颜色可能会影响算法性能。颜色空间缩减即用颜色中具有代表性的一部分表示相近颜色，做法是：将现有颜色空间值除以某个输入值，以获得较少的颜色数。对每个像素进行乘除操作也需要浪费一定的时间（加，减，赋值等代价较低），对于较大的图像，可以预先计算所有可能的值，建立 look up table
该例子没有采用查找表，而是直接对每个像素进行乘除操作，方便理解图像访问方式。
*/
#include <opencv2/opencv.hpp>    
#include<iostream>  
using namespace std;  
using namespace cv;  
void colorReduce(Mat& inputImage, Mat& outputImage, int div);  

int main()  
{  

    Mat srcImage = imread("D:\\vvoo\\lena.jpg");  
    imshow("原始图像", srcImage); 
    Mat dstImage;  
    dstImage.create(srcImage.rows, srcImage.cols, srcImage.type());//效果图的大小、类型与原图片相同   

    //记录起始时间  
    double time0 = static_cast<double>(getTickCount());  
    //调用颜色空间缩减函数  
    colorReduce(srcImage, dstImage, 128);  
    //计算运行时间并输出  
    time0 = ((double)getTickCount() - time0) / getTickFrequency();  
    cout << "此方法运行时间为： " << time0 << "秒" << endl;  //输出运行时间  
    //显示效果图  
    imshow("效果图", dstImage);  
    waitKey(0);  
    return 0;  
}  
//使用指针的方式访问
//void colorReduce(Mat& inputImage, Mat& outputImage, int div){
//  outputImage = inputImage.clone();
//  int rows = outputImage.rows;
//  int cols = outputImage.cols;
//  for (int i = 0; i < rows; ++i){
//      uchar* dataout = outputImage.ptr<uchar>(i);
//      for (int j = 0; j < cols; ++j){
//          dataout[j] = dataout[j] / div*div + div / 2;
//      }
//  }
//}
//使用at方法访问
//void colorReduce(Mat& inputImage, Mat& outputImage, int div){
//  outputImage = inputImage.clone();
//  int rows = outputImage.rows;
//  int cols = outputImage.cols;
//  for (int i = 0; i < rows; ++i){
//      for (int j = 0; j < cols; ++j){
//          outputImage.at<Vec3b>(i, j)[0] = outputImage.at<Vec3b>(i, j)[0] / div*div + div / 2;
//          outputImage.at<Vec3b>(i, j)[1] = outputImage.at<Vec3b>(i, j)[1] / div*div + div / 2;
//          outputImage.at<Vec3b>(i, j)[2] = outputImage.at<Vec3b>(i, j)[2] / div*div + div / 2;
//      }
//  }
//}
//使用迭代器的方式访问
void colorReduce(Mat& inputImage, Mat& outputImage, int div)
//{
//  outputImage = inputImage.clone();
//  //模板必须指明数据类型    
//  Mat_<Vec3b>::iterator it = outputImage.begin<Vec3b>();
//  Mat_<Vec3b>::iterator itend = outputImage.end<Vec3b>();
//  //也可以通过指明cimage类型的方法不写begin和end的类型    
//  /*Mat_<Vec3b> cimage = outputImage;
//  Mat_<Vec3b>::iterator itout =outputImage.begin<Vec3b>();
//  Mat_<Vec3b>::iterator itoutend = cimage.end();*/
//  for (; it != itend; it++/*, it++*/)
//  {
//      (*it)[0] = (*it)[0] / div*div + div / 2;
//      (*it)[1] = (*it)[1] / div*div + div / 2;
//      (*it)[2] = (*it)[2] / div*div + div / 2;
//  }
//}
```

参考博客：   
[http://www.cnblogs.com/Xiaoyan-Li/p/5792796.html](http://www.cnblogs.com/Xiaoyan-Li/p/5792796.html)
[http://blog.csdn.net/qq_29540745/article/details/52443697](http://blog.csdn.net/qq_29540745/article/details/52443697)

