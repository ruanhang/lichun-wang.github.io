---
layout: post
title:  "自定义Caffe Layer"
category: coding
tags: [Caffe]
description: Caffe
---  

### 1.明确自定义层的功能   

* 自定义Layer的实现  
* 自定义一个计算层，实现y=x^power的功能  
* 对自定义层进行测试
* 自定义一个新的输入层

### 2.选择继承的类
任何一个层都可以被继承，然后进行重写函数

![http://img.blog.csdn.net/20170908093920905?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzEyOTQyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast](http://img.blog.csdn.net/20170908093920905?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzEyOTQyNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 3.自定义Layer步骤
步骤：  
1、创建新定义的头文件include/caffe/layers/my_neuron_layer.hpp

* 重新Layer名的方法：virtual inline const char*  type() const { return "MyNeuron"; }
* 如果只是需要cpu方法的话，可以注释掉forward/backward_gpu()这两个方法

2、创建对应src/caffe/src/my_neuron_layer.cpp的源文件  
* 重写方法LayerSetUp,实现从能从prototxt读取参数
* 重写方法Reshape，如果对继承类没有修改的话，就不需要重写
* 重写方法Forward_cpu
* 重写方法Backward_cpu(非必须)
* 如果要GPU支持，则还需要创建src/caffe/src/my_neuron_layer.cu，同理重写Forward_gpu/Backward_gpu(非必须)

3、proto/caffe.proto注册新的Layer  
有三处需要增加
```
          message LayerParameter{
          ...
          ++ optional MyNeuronParameter mysquare_param = 150;
          ...
          }
          ...
          ++ message MyNeuronParameter {
          ++  optional float power = 1 [default = 2];
          ++ }
          ...
          message V1LayerParameter{
          ...
          ++ MYNEURON = 40;
          ...
          }
```
4、my_neuron_layer.cpp添加注册的宏定义

* my_neuron_layer.cpp添加注册的宏定义

```
INSTANTIATE_CLASS(MyneuronLayer);
REGISTER_LAYER_CLASS(MyNeuron);

```

* 如果有my_neuron_layer.cu

```
INSTANTIATE_LAYER_GPU_FUNCS(MyNeuronLayer)
```

5、重新编译install

