---
layout: post
title:  "caffe环境问题以及ubuntu系统问题"
category: coding
tags: [caffe]
description: caffe
---  

转载自：：[http://www.cnblogs.com/empty16/p/4828476.html](http://www.cnblogs.com/empty16/p/4828476.html)  
严正声明：   

* 在linux下面使用命令行操作时，一定要懂得命令行的意思，然后再执行，要不然在不知道接下来会发生什么的情况下输入一通命令，linux很有可能崩掉。  
* 因为在linux下面，使用sudo以及root权限时，是可以对任意一个文件进行操作处理的，即使是正在使用的系统文件。  
* caffe中出现下面这些问题说明在安装过程中有一些步骤没有按照官网说明来，如果按照官网说明一步步安装，一般会一次性通过。  

**Caffe编译问题及解决方案汇总：**  
 在编译caffe代码时，之前的各种错误会显现出来，这时候会出现各种各样的问题：  
**问题1：**  
```
Error: 'make all' 'make test'
.build_release/lib/libcaffe.so: undefined reference to cv::imread(cv::String const&, int)' 
.build_release/lib/libcaffe.so: undefined reference tocv::imencode(cv::String const&, cv::_InputArray const&, std::vector >&, std::vector > const&)'
```
原因：caffe代码中并没有build文件夹，需要新建build文件夹之后再进行编译：   

```python
cd caffe-master　　#打开caffe所在文件夹
cp Makefile.config.example Makefile.config  #change setting in Makefile.config
make all -j8　　#在build文件夹下进行编译
make test -j8 
make runtest -j8　　#使用CPU多核同时进行编译
```

**问题2：**
```
CMake Error at cuda_compile_generated_lrn_layer.cu.o.cmake:206 (message)
```
在成功安装cuda之后，由于路径设置问题，或者路径冲突会产生以下错误，解决方法：  
1.在caffe文件夹下，通过下面该命令查看配置路径：
```
sudo find / -name nvcc
```  
2.通过下面命令查看是否cuda路径冲突：  
```
$PATH
```
如果显示结果有两个cuda环境变量，那么需要移除旧的路径，更新PATH。    
3.重新设置cuda环境变量  
```
在/etc/profile中添加CUDA环境变量　　
PATH=/usr/local/cuda/bin:$PATH  
export PATH
```  
然后注销或重启（因为注销或重启之后PATH会从 ~/.bash_profile文件中重新读取）  

**问题3：pycaffe编译过程中的问题**  
错误信息： 
 
```python
touch python/caffe/proto/__init__.py
CXX/LD -o python/caffe/_caffe.so python/caffe/_caffe.cpp
PROTOC (python) src/caffe/proto/caffe.proto
python/caffe/_caffe.cpp:1:52: fatal error: Python.h: No such file or directory
#include <Python.h>  // NOLINT(build/include_alpha)
compilation terminated.
make: *** [python/caffe/_caffe.so] Error 1
```

因为我的python环境安装的是spyder，而不是Anaconda，因此在makefile.config里面需要对路径进行设置
```python
# Anaconda Python distribution is quite popular. Include path:
# Verify anaconda location, sometimes it's in root.
# ANACONDA_HOME := $(HOME)/anaconda
# PYTHON_INCLUDE := $(ANACONDA_HOME)/include \
# $(ANACONDA_HOME)/include/python2.7 \
# $(ANACONDA_HOME)/lib/python2.7/site-packages/numpy/core/include \

# We need to be able to find libpythonX.X.so or .dylib.
PYTHON_LIB := /usr/lib
# PYTHON_LIB := $(ANACONDA_HOME)/lib
Python.h  can running sudo find / -name 'Python.h' to find the path.
```
 
 
Linux一些常用命令记录及解释：  
1.程序安装  
　　本地安装 命令格式  
```　　
　　sudo dpkg -i softname.deb
```
其中dpkg为Debian Package的缩写，dpkg常用命令有： -i 安装 ；-r 卸载  
dpkg命令是一个底层的安装工具，apt是dpkg上层工具，用于从远程获取软件包以及处理复杂的软件包之间的关系。  
apt常用的用法，apt-get后面接install 或remove对软件进行安装和卸载  

```
　　apt-get install <package>
``` 
 
2.设置系统root密码   
如果使用光盘安装Ubuntu，按照安装向导来进行帐号、分区等设置，而在这个安装向导程序中没有提示进行root密码的设置，所以在 Ubuntu安装好后需要手动设置root密码。而如果是跳过安装向导，点击桌面上的Install图标来进行安装的话，在安装过程中则会提示设置 root密码。当然，如果需要修改root密码也可以使用以下方法：打开终端，在终端中输入命令：  

```  
sudo passwd root
```
接下来，按照提示一步步设置系统的root密码。  
输入新的 UNIX 口令：  
重新输入新的 UNIX 口令：  
passwd：已成功更新密码  
口令：  
 
3.删除文件夹和文件  
在ubuntu里面有些文件夹通过右键方式无法删除，这时候就需要使用命令来进行删除。  
删除文件：  
```
cd /usr/local/src      #打开文件所在位置
sudo rm ./file-name　　#删除文件
```
删除文件夹：  
```
cd /usr/local/src
sudo rm -r ./folder-name
```  
 
3. 使用命令更改文件或文件夹名  
```
sudo mv 旧文件名 新文件名
```
 
4. 使用显示内核版本  
```
uname a
```

5.Ubuntu批量复制删除文件命令  
批量复制文件命令  
```
sudo cp -R /srcfolder/*  /dstfolder
```
批量删除文件命令：  
```
sudo rm -rf /usr/share/stardict/dic/*
```
 
 
 
Ubuntu14.04使用中的一些问题及解决方法：  
1.内核出现问题时解决方法  
该问题困扰我很久，因为dpkg时程序安装命令，该命令出问题导致新的软件不能安装，非常烦，查了很多资料，终于找到解决方法。  
错误提示：  dpkg: 在处理时有错误发生： 
```
 linux-image-extra-3.19.0-28-generic 
 linux-image-3.19.0-28-generic
```
解决思路：将dpkg包中的信息先备份，在新的info信息复制到文件夹中更新内核
```c++
sudo mv /var/lib/dpkg/info /var/lib/dpkg/info_old    　　//现将info文件夹更名
sudo mkdir /var/lib/dpkg/info   　　　　　　　　　　　　　　//再新建一个新的info文件夹
sudo apt-get update　　　　　　　　　　　　　　　　　　　　　 //更新源
sudo apt-get -f install 　　　　　　　　　 　　　　　　　　　
sudo mv /var/lib/dpkg/info/* /var/lib/dpkg/info_old  　　//将info中文件全部移到info_old文件夹下
sudo rm -rf /var/lib/dpkg/info 　　　　　　　　　　　　　　　//把自己新建的info文件夹删掉
sudo mv /var/lib/dpkg/info_old /var/lib/dpkg/info 　　　　//把以前的info文件夹重新改回名字
```
通过上述命令可以解决内核移除失败，更新问题。  
 
2.Ubuntu14.04 无法识别硬盘exfat分区  
为什么使用exfat格式呢？主要有以下两种原因：  
1、三大主流操作系统（Linux、Mac、Windows）都支持exfat格式。  
2、exfat支持大于4G的文件。  
在ubuntu下，由于版权的原因（据说），默认不支持exfat格式的u盘，不过可以很方便就能添加对exfat的支持：  
1、对于ubuntu 14.04版本，直接运行下面的命令就可以了：  
```　　 
sudo apt-get install exfat-utils
```
安装完之后重启生效。（如果不重启不行，则重启）  
 
3.设置PYTHON路径方法  
```
export PYTHONPATH=/home/username/caffe/python
```
查看路径  
```
echo $PYTHONPATH
```
 
4.Ubuntu不能对exfat以及ntfs等格式磁盘写入文件  
问题描述：
```
the disk for the xx is not ready yet or not present  
　　　　　acpi pcc probe failed
```
解决方式：  
```
sudo su #获取root权限
mount -o remount,rw /
dpkg --configure -a
```
 
5.caffe的python接口配置问题  
在使用make pycaffe -j8命令完成caffe的python接口生成之后，还需要将python接口的路径进行设置。  
路径设置一般有两种方式（具体方法百度），为方便使用，这里设置为永久路径。  
使用命令  
```
gedit ~/.bashrc
```
来对路径进行设置，在文件最后一行加入路径：  
```
export PYTHONPATH=/home/startag/caffe/python/:/home/startag/caffe/python/caffe/
```
注销或者重启，路径生效。  
 
import caffe时错误提示  
1. 错误提示：ImportError: No module named skimage.io  
解决方法：  
直接使用终端安装：  
```
pip install -U scikit-image
```
如果提示不识别   pip  命令，在Ubuntu14.04（64bit）下，使用下面命令安装pip包管理软件，也可以使用新立得软件包搜索“scikit-image”安装。  
```
wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
sudo python get-pip.py
```
 
问题：
```
ImportError: No module named google.protobuf.internal
```
提示错误可使用新立得软件包搜索“python-protobuf”安装。  
然后使用import caffe测试接口是否调试成功。  
 
问题：
```
from google.protobuf import symbol_database as _symbol_database
ImportError: cannot import name symbol_database
```
解决方法： 
```
sudo pip install --upgrade protobuf
```
 
 
 
6. caffe中的python接口和matlab接口配置及常见问题汇总：   
在配置好了caffe环境之后，我们需要使用到caffe中的接口。caffe的接口分为3种，cmd接口，matlab接口和python接口。  
cmd接口在使用make all -j8过程中已经生成，位置在tools里面。而matlab接口特别是python接口需要配置，期间还会遇到各种各样的问题。  
在对caffe的matlab和python接口进行编译时可能会遇到g++版本过高问题，解决方法：Caffe使用：安装gcc4.7和g++4.7。  
在make pycaffe之后，需要使用make dist来将生成的python文件进行整理并设置caffe路径。  
在～/.bashrc文件中加入路径：（ 问题：ImportError: libcaffe.so: cannot open shared object file: No such file or directory解决方法）  
#多个路径使用：分割开  
```
export LD_LIBRARY_PATH=/opt/intel/mkl/lib/intel64:/usr/local/cuda/lib64:/home/startag/caffe/distribute/lib  
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
export PYTHONPATH=/home/startag/caffe/distribute/python:$PYTHONPATH
```
python接口配置按照caffe官网interface中步骤进行，在～/.bashrc文件中写入PYTHONPATH路径，具体见官网。
遇见的问题解决方法：方法1   
**建议：在使用caffe时候，确定一个版本，然后把路径写入～/.bashrc 文件中。当然，也可以使用多个版本，不过需要把每一个版本的路径都要加入到～/.bashrc文件中，比较麻烦，如果自己需要使用caffe，使用软连接方法建立与caffe的软连接。  
方法：  
```
ln -s caffe-root 目标文件夹
 构建fast-rcnn时提示：OpenCV - cannot find module cv2
解决方法
```
&. 使用draw_net.py绘制网络结构方法：  
使用draw_net.py绘制网络结构时提示错误信息：  
```
permission denied: 
```
解决方法：让该文件具有系统权限  
```
chmod u+x ./python/draw_net.py
```
 
出现下面错误时说明系统已经严重损坏，不保证可以完全修复  
1.误将cuda卸载之后，cuda-driver包损坏时的解决方案：  
使用 aptitude进行安装（这个经测试不好用，这样安装可能是非官方驱动）  
```
Install aptitude
sudo apt-get install aptitude
Install main package
sudo aptitude install cuda
```
 
2. caffe中安装build-essential提示包损坏解决方法  
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential -f
```
 
当因安装版本问题出现错误时，如本来应该在Ubuntu14.04上安装cuda7.0，但是错误的在Ubuntu15.04上安装（应该安装对应的7.5），提示一下错误时：  
```
dpkg: error processing archive  (--install):cuda-repo-ubuntu1504_7.5-18_amd64.deb
trying to overwrite '/etc/apt/sources.list.d/cuda.list',
which is also in package cuda-repo-ubuntu1404_7.0-18_amd64.deb
```
 
解决方案：  
```
sudo dpkg -i --force-overwrite cuda-repo-ubuntu1504_7.5-18_amd64.deb
```
 

遇到该问题时：
```
ImportError: /usr/lib/liblapack.so.3: undefined symbol: ATL_chemv
```

解决方法：
```
http://stackoverflow.com/questions/8917977/installing-lapack-for-numpy
```