---
layout: post
title:  "caffe自定义Neuron层"
category: [coding ]
tags: [c++,caffe,algorithms,2017]
description: caffe自定义Neuron层"
header-img: "img/pages/template.jpg"
---

### 说明
在使用caffe的过程中，如何设计自己的layer，实现caffe所没有的功能呢？
在自定义caffe的layer的时候，如下几个layer是比较常见的：

* 1、DataLayer 自定义输入层网络
* 2、NeuronLayer 自定义神经层，中间的运算
* 3、LossLayer 自定义损失函数

下面我们就以**y = x^2**为例子简单介绍一下如何自定义NeuronLayer层。

### 1、创建新定义的头文件include/caffe/layers/my_neuron_layer.hpp
可以仿照已经定义好的**neuron_layer**进行修改,如：log_layer

* 1.首先修改#ifndef,#define
	#ifndef CAFFE_MY_NEURON_LAYER_HPP_
	#define CAFFE_MY_NEURON_LAYER_HPP_

* 2.新Layer名的方法：

	virtual inline const char*  type() const { return "MyNeuron"; }

* 3.如果只是需要**cpu**方法的话，可以注释掉forward/backward_gpu()这两个方法


### 2. 创建对应src/caffe/src/my_neuron_layer.cpp的源文件
* 1.重写方法LayerSetUp,实现从能从prototxt读取参数
* 2.重写方法Reshape，如果对继承类没有修改的话，就不需要重写
* 3.重写方法Forward_cpu
* 4.重写方法Backward_cpu(非必须)
* 5.如果要GPU支持，则还需要创建src/caffe/src/my_neuron_layer.cu，同理重写方法Forward_gpu/Backward_gpu(非必须)

### 3. proto/caffe.proto注册新的Layer,在下面相应地方，添加

```
message LayerParameter{

  optional MyNeuronParameter my_neuron_param = 150;

}

message MyNeuronParameter {
  optional float power = 1 [default = 2];
}

message V1LayerParameter{

  MYNEURON = 40;

}
```

### 4. my_neuron_layer.cpp添加注册的宏定义,在第2部中最下面，**其中REGISTER_LAYER_CLASS中变量没有Layer，一定要注意**
	INSTANTIATE_CLASS(MyNeuronLayer);
	REGISTER_LAYER_CLASS(MyNeuron);

### 5. 重新编译和install
* 1.cd 到build目录下
cmake -D CPU_ONLY = ON -D CMAKE_PREFIX_INSTALL=/usr/local ..
* 2. cd ..
make all


### 6. 测试代码
下面的代码是测试deploy文件以及调用的python程序，例子图片在[网盘链接](http://pan.baidu.com/s/1jH4i6qU)中,密码为：4zuy，网盘中还含有全部的修改代码，欢迎下载查看。

``` c++
//deploy文件
name: "CaffeNet"
input: "data"
input_shape {
  dim: 1 # batchsize
  dim: 1 # number of colour channels - rgb
  dim: 28 # width
  dim: 28 # height
}

layer {
  name: "myneuron"
  type: "MyNeuron"
  bottom: "data"
  top: "data_out"
  my_neuron_param {
    power : 2
  }
}

//python代码
# -*- coding: utf-8 -*-

import numpy as np
import matplotlib.pyplot as plt
import os
import sys
import caffe

deploy_file = "./deploy.prototxt"
test_data   = "./5.jpg"

if __name__ == '__main__':
  
  net = caffe.Net(deploy_file,caffe.TEST)

  transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})

  transformer.set_transpose('data', (2, 0, 1))

  img = caffe.io.load_image(test_data,color=False)

  net.blobs['data'].data[...] = transformer.preprocess('data', img)

  print net.blobs['data'].data[0][0][14]

  out = net.forward()

  print out['data_out'][0][0][14]
```




### 7. 修改的代码

```c++
// my_neuron_layer.hpp
#ifndef CAFFE_MY_NEURON_LAYER_HPP_
#define CAFFE_MY_NEURON_LAYER_HPP_

#include <vector>

#include "caffe/blob.hpp"
#include "caffe/layer.hpp"
#include "caffe/proto/caffe.pb.h"

#include "caffe/layers/neuron_layer.hpp"

namespace caffe {

template <typename Dtype>
class MyNeuronLayer : public NeuronLayer<Dtype> {
 public:
  
  explicit MyNeuronLayer(const LayerParameter& param)
      : NeuronLayer<Dtype>(param) {}
  virtual void LayerSetUp(const vector<Blob<Dtype>*>& bottom,
      const vector<Blob<Dtype>*>& top);

  virtual inline const char* type() const { return "MyNeuron"; }

 protected:
  
  virtual void Forward_cpu(const vector<Blob<Dtype>*>& bottom,
      const vector<Blob<Dtype>*>& top);
  virtual void Forward_gpu(const vector<Blob<Dtype>*>& bottom,const vector<Blob<Dtype>*>& top);

  virtual void Backward_cpu(const vector<Blob<Dtype>*>& top,
      const vector<bool>& propagate_down, const vector<Blob<Dtype>*>& bottom);
  virtual void Backward_gpu(const vector<Blob<Dtype>*>& top,const vector<bool>& propagate_down, const vector<Blob<Dtype>*>& bottom);

  Dtype power_;
};

}  // namespace caffe

#endif  // CAFFE_MY_NEURON_LAYER_HPP_
```


``` c++
//my_neuron_layer.cpp
#ifdef USE_OPENCV
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

#include <fstream>  // NOLINT(readability/streams)
#include <iostream>  // NOLINT(readability/streams)
#include <string>
#include <utility>
#include <vector>
#include <stdio.h>

#include "caffe/data_transformer.hpp"
#include "caffe/layers/base_data_layer.hpp"
#include "caffe/layers/my_data_layer.hpp"
#include "caffe/util/benchmark.hpp"
#include "caffe/util/io.hpp"
#include "caffe/util/math_functions.hpp"
#include "caffe/util/rng.hpp"

namespace caffe {

template <typename Dtype>
MyDataLayer<Dtype>::~MyDataLayer<Dtype>() {
  this->StopInternalThread();
}

template <typename Dtype>
void MyDataLayer<Dtype>::DataLayerSetUp(const vector<Blob<Dtype>*>& bottom,const vector<Blob<Dtype>*>& top){
    // Init private variables.
    image_address_ = this->layer_param_.my_data_param().image_address();
    start_col_= this->layer_param_.my_data_param().start_col();
    end_col_  = this->layer_param_.my_data_param().end_col();
    sample_width_ = this->layer_param_.my_data_param().sample_width();
    sample_height_= this->layer_param_.my_data_param().sample_height();
    
    lines_id_ = 0;

    int label,x,y;
    cv::Mat image;
    source_image_ = cv::imread(image_address_);
    for(int i=0 ; i< 50 ; i++){
      label = (int)(i/5);
      for(int j=start_col_ ; j<end_col_ ; j++){
         y=20*i;
         x=20*j;
         image = get_one_sample(source_image_,x,y,20,20);
         samples_.push_back(std::make_pair(image,label));
      }
    }

    CHECK(!samples_.empty()) << "Data is empty";

    // Shuffle images
    if(this->layer_param_.my_data_param().shuffle()){
	    LOG(INFO) << "Shuffling data";
	    ShuffleImages();
    }


    // Save images
    if(this->layer_param_.my_data_param().is_save()){
	LOG(INFO) << "Saving images ";
        for(int i=0;i<samples_.size();i++){
  	    string save_folder = this->layer_param_.my_data_param().save_folder();
	    char img_name[128];
	    sprintf(img_name,"%s/%d.jpg",save_folder.c_str(),i);
	    cv::imwrite(img_name,samples_[i].first);
        }
    }


    // Read an image, and use it to initialize the top blob
    cv::Mat cv_img = samples_[lines_id_].first;
    CHECK(cv_img.data) << "Could not load first image";

    // Use data_transformer to infer the expected blob shape from a cv_image
    vector<int> top_shape = this->data_transformer_->InferBlobShape(cv_img);
    this->transformed_data_.Reshape(top_shape);

    //Reshape prefetch_data and top[0] according to the batch_size.
    const int batch_size = this->layer_param_.my_data_param().batch_size();
    top_shape[0] = batch_size;
    for(int i=0;i<this->PREFETCH_COUNT;++i){
	    this->prefetch_[i].data_.Reshape(top_shape);
    }
    top[0]->Reshape(top_shape);

    LOG(INFO) << "output data size: " << top[0]->num() << "," << top[0]->channels() << "," << top[0]->height() << "," << top[0]->width();

    // label
    vector<int> label_shape(1,batch_size);
    top[1]->Reshape(label_shape);
    for(int i=0;i<this->PREFETCH_COUNT;++i){
	    this->prefetch_[i].label_.Reshape(label_shape);
    }


}


template <typename Dtype>
void MyDataLayer<Dtype>::ShuffleImages(){
    std::srand ( unsigned ( std::time(0) ) );
    std::random_shuffle(samples_.begin(),samples_.end());
}

template <typename Dtype>
void MyDataLayer<Dtype>::load_batch(Batch<Dtype>* batch){
    CPUTimer batch_timer;
    batch_timer.Start();
    double read_time = 0;
    double trans_time = 0;
    CPUTimer timer;
    CHECK(batch->data_.count());
    CHECK(this->transformed_data_.count());

    MyDataParameter my_data_param = this-> layer_param_.my_data_param();
    // Get batch size
    const int batch_size = my_data_param.batch_size();

    // Reshape according to the first image of each batch
    // on single input batches allows for inputs of varying dimension
    cv::Mat cv_img = samples_[lines_id_].first;
    CHECK(cv_img.data) << "Could not load "<<lines_id_<<" sample";
    // Use data_transformer to infer the expected blob shape from a cv_img
    vector<int> top_shape = this->data_transformer_->InferBlobShape(cv_img);
    this->transformed_data_.Reshape(top_shape);
    // Reshape batch according to the batch_size
    top_shape[0] = batch_size;
    batch->data_.Reshape(top_shape);
    
    Dtype* prefetch_data = batch->data_.mutable_cpu_data();
    Dtype* prefetch_label= batch->label_.mutable_cpu_data();

    // datum scales
    int samples_size = samples_.size();
    for(int item_id=0;item_id<batch_size;++item_id){
      // get a blob
      timer.Start();
      CHECK_GT(samples_size, lines_id_);
      cv::Mat sample = samples_[lines_id_].first;
      CHECK(sample.data) << "Could not load "<<lines_id_<<" sample";
      read_time += timer.MicroSeconds();
      timer.Start();
      // apply transformations to the image
      int offset = batch->data_.offset(item_id);
      this->transformed_data_.set_cpu_data(prefetch_data + offset);
      this->data_transformer_->Transform(sample,&(this->transformed_data_));
      trans_time += timer.MicroSeconds();

      prefetch_label[item_id] = samples_[lines_id_].second;
      // got the the next iter
      lines_id_++;
      if(lines_id_>=samples_size){
              // We have reached the end. restart from the first.
	      DLOG(INFO) << "Restarting data prefetching from start.";
	      lines_id_=0;
	      if(my_data_param.shuffle()){
		      ShuffleImages();
	      }
      }
    }
    batch_timer.Stop();
    DLOG(INFO) << "Prefetch batch: " << batch_timer.MilliSeconds() << " ms.";
    DLOG(INFO) << "     Read time: " << read_time / 1000 << " ms.";
    DLOG(INFO) << "Transform time: " << trans_time / 1000 << " ms.";
}

INSTANTIATE_CLASS(MyDataLayer);
REGISTER_LAYER_CLASS(MyData);

} // namespaces caffe
#endif  // USE_OPENCV

```

共同学习，共同进步。
  



