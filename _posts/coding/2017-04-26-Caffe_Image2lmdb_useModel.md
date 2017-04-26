---
layout: post
title:  "caffe将图像转化为训练集以及使用caffe模型"
category: [coding ]
tags: [c++,caffe,algorithms]
description: caffe将图像转化为训练集以及使用caffe模型
header-img: "img/pages/template.jpg"
---

### 一、生成MLDB数据库
* 1、将图片放入不同的文件夹：如：train/class1, train/class2 , test/class3, test/class4
* 2、生成train.txt以及test.txt文件存放图像名称及类别
	* cd 到train ， test同层文件目录
	* ls 【文件路径：train/1】 | sed "s:^:【文件路径1/】:"| sed "s:$:  【类别1:】" >> t_train.txt
	* ls train/first | sed "s:^:first/:"| sed "s:$: 1:" >> t_train.txt
 
* 3、调用convert_imageset命令：
	* cd 到train ， test同层文件目录
    * convert_imageset --resize_width=40 --resize_height=40 【文件路径名到class文件 上一层】 【txt描述文件名】【lmdb文件名】
    * convert_imageset --resize_width=40 --resize_height=40 /home/leospring/caffe-master/.build_	release/teachExample/3/test/ ./t_test.txt ./t_test_lmdb
* 3.1、**convert_imageset命令详细介绍**
	* convert_imageset [FLAGS] [ROOTFOLDER/] [LISTFILE] [DB_NAME]   需要带四个参数：
	>* FLAGS: 图片参数组    
	> 1、gray: 是否以灰度图的方式打开图片。程序调用opencv库中的imread()函数来打开图片，默认为false   
	>2、-shuffle: 是否随机打乱图片顺序。默认为false   
	>3、-backend:需要转换成的db文件格式，可选为leveldb或lmdb,默认为lmdb    
	>4、resize_width/resize_height: 改变图片的大小。在运行中，要求所有图片的尺寸一致，因此需要改变图片大小。 程序调用opencv库的resize（）函数来对图片放大缩小，默认为0，不改变    >5、-check_size: 检查所有的数据是否有相同的尺寸。默认为false,不检查   
    >6、-encoded: 是否将原图片编码放入最终的数据中，默认为false    
    >7、-encode_type: 与前一个参数对应，将图片编码为哪一个格式：‘png','jpg'......
    * ROOTFOLDER/: 图片存放的绝对路径，从linux系统根目录开始
    * LISTFILE: 图片文件列表清单，一般为一个txt文件，一行一张图片
    * DB_NAME: 最终生成的db文件存放目录


### 二、使用训练好的模型
* 1、均值文件mean file
>将所有训练样本的均值保存为文件
>图片减去均值后，再进行训练和测试，会提高速度和精度
>运行方法：（使用Caffe工具）**compute_image_mean [train_lmdb] [mean.binaryproto]**

* 2、改写deploy文件
* 1. 把数据层（Data Layer）和连接数据层的Layers去掉(即top:data的层)

~~~
input: "data"
input_shape {
dim: 1 # batchsize,每次forward的时候输入的图片个数
dim: 3 # number of colour channels - rgb
dim: 28 # width
dim: 28 # height
    }
~~~

* 2. 去掉输出层和连接输出层的Layers（即bottom:label）

~~~
layer {
  name: "prob"
  type: "Softmax"
  bottom: "ip2"
  top: "prob"
}
~~~

3、调用:参考炼石成金课程，利用lenet识别mnist数据

``` c++
#include "opencv2/dnn.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"
using namespace cv;
using namespace cv::dnn;
#include <fstream>
#include <iostream>
#include <cstdlib>
using namespace std;
/* Find best class for the blob (i. e. class with maximal probability) */ 
void getMaxClass(dnn::Blob &probBlob, int *classId, double *classProb)
{
    Mat probMat = probBlob.matRefConst().reshape(1, 1); //reshape the blob to 1x1000 matrix
    Point classNumber;
    minMaxLoc(probMat, NULL, classProb, NULL, &classNumber);
    *classId = classNumber.x;
}
int main(int argc,char* argv[]){
    String modelTxt = "mnist_deploy.prototxt";
    String modelBin = "lenet_iter_10000.caffemodel";
    String imageFile = (argc > 1) ? argv[1] : "5.jpg";
    //! [Create the importer of Caffe model] 导入一个caffe模型接口 
    Ptr<dnn::Importer> importer; 
    importer = dnn::createCaffeImporter(modelTxt, modelBin); 
    if (!importer){
        std::cerr << "Can't load network by using the following files: " << std::endl;
        std::cerr << "prototxt:   " << modelTxt << std::endl;
        std::cerr << "caffemodel: " << modelBin << std::endl;
        exit(-1);
    }
    //! [Initialize network] 通过接口创建和初始化网络
    Net net;
    importer->populateNet(net);  
    importer.release();

    //! [Prepare blob] 读取一张图片并转换到blob数据存储
    Mat img = imread(imageFile,0); //[<Important>] "0" for 1 channel, Mnist accepts 1 channel
    if (img.empty())
    {
        std::cerr << "Can't read image from the file: " << imageFile << std::endl;
        exit(-1);
    }
    resize(img, img, Size(28, 28));                   //[<Important>]Mnist accepts only 28x28 RGB-images
    dnn::Blob inputBlob = cv::dnn::Blob(img);   //Convert Mat to dnn::Blob batch of images
    //! [Set input blob] 将blob输入到网络
    net.setBlob(".data", inputBlob);        //set the network input

    //! [Make forward pass] 进行前向传播
    net.forward();                          //compute output

    //! [Gather output] 获取概率值
    dnn::Blob prob = net.getBlob("prob");   //[<Important>] gather output of "prob" layer
    int classId;
    double classProb;
    getMaxClass(prob, &classId, &classProb);//find the best class
    //! [Print results] 输出结果
    std::cout << "Best class: #" << classId << "'" << std::endl;
    std::cout << "Probability: " << classProb * 100 << "%" << std::endl;    
    return 0;
}
```


  



