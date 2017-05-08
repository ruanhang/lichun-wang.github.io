---
layout: post
title:  "caffe自定义Neuron层"
category: [coding ]
tags: [c++,caffe,algorithms]
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
#include <vector>

#include "caffe/layers/my_neuron_layer.hpp"
#include "caffe/util/math_functions.hpp"

namespace caffe {

template <typename Dtype>
void MyNeuronLayer<Dtype>::LayerSetUp(const vector<Blob<Dtype>*>& bottom,const vector<Blob<Dtype>*>& top){
  
  NeuronLayer<Dtype>::LayerSetUp(bottom,top);
  power_ = this->layer_param_.my_neuron_param().power();

}

// Compute y = x^power
template <typename Dtype>
void MyNeuronLayer<Dtype>::Forward_cpu(const vector<Blob<Dtype>*>& bottom,const vector<Blob<Dtype>*>& top){

  Dtype* top_data = top[0]->mutable_cpu_data();
  const int count = bottom[0]->count();
  caffe_powx(count, bottom[0]->cpu_data(), Dtype(power_), top_data);
}

template <typename Dtype>
void MyNeuronLayer<Dtype>::Backward_cpu(const vector<Blob<Dtype>*>& top,const vector<bool>& propagate_down,const vector<Blob<Dtype>*>& bottom){
  const int count = top[0]->count();
  const Dtype* top_diff = top[0]->cpu_diff();
  if(propagate_down[0]){
    const Dtype* bottom_data = bottom[0]->cpu_data();
    Dtype* bottom_diff = bottom[0]->mutable_cpu_diff();
    caffe_powx(count, bottom_data, Dtype(power_ - 1), bottom_diff);
    caffe_scal(count, Dtype(power_), bottom_diff);
    caffe_mul(count, bottom_diff, top_diff, bottom_diff);
  }

}

#ifdef CPU_ONLY
STUB_GPU(MyNeuronLayer);
#endif

INSTANTIATE_CLASS(MyNeuronLayer);
REGISTER_LAYER_CLASS(MyNeuron);

}// namespace caffe


```

共同学习，共同进步。
  



