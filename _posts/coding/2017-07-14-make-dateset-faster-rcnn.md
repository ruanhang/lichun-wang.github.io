---
layout: post
title:  "Ｃ++--this指针"
category: coding
tags: [ｃ++]
description: c++
---  

### 0.前言

在Faster-rcnn上进行自己的实验，一般有两种方法：   
1）. 改动源码，适合自己的数据集格式   
2）. 将自己的数据制作成voc格式，直接套用   
我在自己的实验中，选择了第二种方式，在此记录整理一下数据集的制作步骤。   
voc数据集有三个文件夹：  

Annonations:存放xml文件  
JPEGImages:存放图片images  
ImageSets:存放txt文件   
制作的流程是：图片名和GT写入txt –> txt转成xml –> xml转成ImageSets中四个txt  
1.JPEGImages图片写入txt  

由于我的训练图片和其对应的ground truth存放在多级目录下，需要自己写代码将多级目录下的图片读取并重新命名成voc格式：如“000005.jpg”。因此，不需要在自己标注GT，只需读取写入即可。   
如需手动画包围框可参考：http://blog.csdn.net/sinat_30071459/article/details/50723212  

```
#include <stdio.h>
#include <stdlib.h>
#include <string>
#include <dirent.h>
#include <unistd.h>
#include <fstream>
#include <opencv2/opencv.hpp>
using namespace std;
using namespace cv;

void changeFold();
void bornTxt();

fstream file;
int readFileList(char *basePath)
{
    DIR *dir;
    struct dirent *ptr;
    char base[1000];
    if ((dir=opendir(basePath)) == NULL)
    {
        perror("Open dir error...");
        exit(1);
    }

    while ((ptr=readdir(dir)) != NULL)
    {
        if(strcmp(ptr->d_name,".")==0 || strcmp(ptr->d_name,"..")==0)    ///current dir OR parrent dir
            continue;
        else if(ptr->d_type == 8)    ///file
            {
               printf("d_name:%s/%s\n",basePath,ptr->d_name);
               file<<basePath<<'/'<<ptr->d_name;
               char basePath2[1000];
                strcpy(basePath2,basePath) ;
               basePath2[52] = 'l';
               basePath2[53] = 'a';
               basePath2[54] = 'b';
               basePath2[55] = 'e';
               basePath2[56] = 'l';
               strcat(basePath2,"/");
               strcat(basePath2,ptr->d_name);
               fstream file2;
       basePath2[strlen(basePath2)-3] = 't';
       basePath2[strlen(basePath2)-2] = 'x';
       basePath2[strlen(basePath2)-1] = 't';
       file2.open(basePath2,ios::in);
       int temp=0;
       file2>>temp;
       file2>>temp;
       file2>>temp;
       file<<' '<<temp;
       file2>>temp;
       file<<' '<<temp;
       file2>>temp;
       file<<' '<<temp;
       file2>>temp;
       file<<' '<<temp<<"\n";
       file2.close();
            }
        else if(ptr->d_type == 10)    ///link file
            {
              printf("d_name:%s/%s\n",basePath,ptr->d_name);
              file<<basePath<<'/'<<ptr->d_name<<"\n";
              }
        else if(ptr->d_type == 4)    ///dir
        {
            memset(base,'\0',sizeof(base));
            strcpy(base,basePath);
            strcat(base,"/");
            strcat(base,ptr->d_name);
            readFileList(base);
        }
    }
    closedir(dir);
    return 1;
}
void bornTxt()
{
    file.open("/home/ruanhang/caffe-master/data/CompCars/data/data/images.txt",ios::out);
    DIR *dir;
    char basePath[1000] = "/home/ruanhang/caffe-master/data/CompCars/data/data/image";
    printf("the current dir is : %s\n",basePath);
    readFileList(basePath);
    return ;
}

int main(void)
{
  // bornTxt();
   changeFold();
   return 0;
}
void changeFold()
{
   //打开原txt文件
    string fileName = "//home//ruanhang//caffe-master//data//CompCars//data//data//images.txt";
    fstream file;
    file.open(fileName.c_str(),ios::in);
    if(!file)
       exit(0);
    //打开写入的txt文件
     string txtName = "//home//ruanhang//py-faster-rcnn//data//VOCdevkit2007//VOC2007//JPEGImages//car_GT.txt";
       fstream fileTXT;
       fileTXT.open(txtName.c_str(),ios::out);
       if(!fileTXT)
          exit(0);
   string  eachFileName;
   int cordinator[4];
   int baseNum = 100000;
   while(!file.eof())
   {
      eachFileName.clear();
      string writePath = "//home//ruanhang//py-faster-rcnn//data//VOCdevkit2007//VOC2007//JPEGImages//";
      /////读取文件内容
       file>>eachFileName;
       file>>cordinator[0];
       file>>cordinator[1];
       file>>cordinator[2];
       file>>cordinator[3];

       cv::Mat img;
       img = cv::imread(eachFileName);
       char strNum[20];
       sprintf(strNum,"%d",baseNum);
       strNum[0] -= 1;
       cout<<baseNum<<endl;
       baseNum++;
       writePath += strNum;
       writePath += ".jpg";
       imwrite(writePath,img);
       fileTXT<<strNum<<".jpg";
       fileTXT<<" ";
       fileTXT<<"car"<<" ";
       fileTXT<<cordinator[0]<<" "<<cordinator[1]<<" "<<cordinator[2]<<" "<<cordinator[3]<<endl;
   }

}
```

### 2.txt转成xml，存入Annonations

```
clc;
clear;
imgpath='JPEGImages/';
txtpath='JPEGImages/car_GT.txt';
xmlpath_new='Annotations/';
foldername='VOC2007';
fidin=fopen(txtpath,'r');
lastname='begin';
while ~feof(fidin)
     tline=fgetl(fidin);
     str = regexp(tline, ' ','split');
     filepath=[imgpath,str{1}];
     img=imread(filepath);
     [h,w,d]=size(img);
      imshow(img);
     % rectangle('Position',[str2double(str{3}),str2double(str{4}),str2double(str{5})-str2double(str{3}),str2double(str{6})-str2double(str{4})],'LineWidth',4,'EdgeColor','r');
     % pause(0.1);      
        if strcmp(str{1},lastname)
           object_node=Createnode.createElement('object');
           Root.appendChild(object_node);
           node=Createnode.createElement('name');
           node.appendChild(Createnode.createTextNode(sprintf('%s',str{2})));
           object_node.appendChild(node);

           node=Createnode.createElement('pose');
           node.appendChild(Createnode.createTextNode(sprintf('%s','Unspecified')));
           object_node.appendChild(node);

           node=Createnode.createElement('truncated');
           node.appendChild(Createnode.createTextNode(sprintf('%s','0')));
           object_node.appendChild(node);

           node=Createnode.createElement('difficult');
           node.appendChild(Createnode.createTextNode(sprintf('%s','0')));
           object_node.appendChild(node);

           bndbox_node=Createnode.createElement('bndbox');
           object_node.appendChild(bndbox_node);

           node=Createnode.createElement('xmin');
           node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{3}))));
           bndbox_node.appendChild(node);

           node=Createnode.createElement('ymin');
           node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{4}))));
           bndbox_node.appendChild(node);

           node=Createnode.createElement('xmax');
           node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{5}))));
           bndbox_node.appendChild(node);

           node=Createnode.createElement('ymax');
           node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{6}))));
           bndbox_node.appendChild(node);
        else 
           %copyfile(filepath, 'JPEGImages');
           if exist('Createnode','var')
              tempname=lastname;
              tempname=strrep(tempname,'.jpg','.xml');
              xmlwrite(tempname,Createnode);   
           end


            Createnode=com.mathworks.xml.XMLUtils.createDocument('annotation');
            Root=Createnode.getDocumentElement;
            node=Createnode.createElement('folder');
            node.appendChild(Createnode.createTextNode(sprintf('%s',foldername)));
            Root.appendChild(node);
            node=Createnode.createElement('filename');
            node.appendChild(Createnode.createTextNode(sprintf('%s',str{1})));
            Root.appendChild(node);
            source_node=Createnode.createElement('source');
            Root.appendChild(source_node);
            node=Createnode.createElement('database');
            node.appendChild(Createnode.createTextNode(sprintf('My Database')));
            source_node.appendChild(node);
            node=Createnode.createElement('annotation');
            node.appendChild(Createnode.createTextNode(sprintf('VOC2007')));
            source_node.appendChild(node);

           node=Createnode.createElement('image');
           node.appendChild(Createnode.createTextNode(sprintf('flickr')));
           source_node.appendChild(node);

           node=Createnode.createElement('flickrid');
           node.appendChild(Createnode.createTextNode(sprintf('NULL')));
           source_node.appendChild(node);
           owner_node=Createnode.createElement('owner');
           Root.appendChild(owner_node);
           node=Createnode.createElement('flickrid');
           node.appendChild(Createnode.createTextNode(sprintf('NULL')));
           owner_node.appendChild(node);

           node=Createnode.createElement('name');
           node.appendChild(Createnode.createTextNode(sprintf('xiaoxianyu')));
           owner_node.appendChild(node);
           size_node=Createnode.createElement('size');
           Root.appendChild(size_node);

          node=Createnode.createElement('width');
          node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(w))));
          size_node.appendChild(node);

          node=Createnode.createElement('height');
          node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(h))));
          size_node.appendChild(node);

         node=Createnode.createElement('depth');
         node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(d))));
         size_node.appendChild(node);

          node=Createnode.createElement('segmented');
          node.appendChild(Createnode.createTextNode(sprintf('%s','0')));
          Root.appendChild(node);
          object_node=Createnode.createElement('object');
          Root.appendChild(object_node);
          node=Createnode.createElement('name');
          node.appendChild(Createnode.createTextNode(sprintf('%s',str{2})));
          object_node.appendChild(node);

          node=Createnode.createElement('pose');
          node.appendChild(Createnode.createTextNode(sprintf('%s','Unspecified')));
          object_node.appendChild(node);

          node=Createnode.createElement('truncated');
          node.appendChild(Createnode.createTextNode(sprintf('%s','0')));
          object_node.appendChild(node);

          node=Createnode.createElement('difficult');
          node.appendChild(Createnode.createTextNode(sprintf('%s','0')));
          object_node.appendChild(node);

          bndbox_node=Createnode.createElement('bndbox');
          object_node.appendChild(bndbox_node);

         node=Createnode.createElement('xmin');
         node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{3}))));
         bndbox_node.appendChild(node);

         node=Createnode.createElement('ymin');
         node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{4}))));
         bndbox_node.appendChild(node);

        node=Createnode.createElement('xmax');
        node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{5}))));
        bndbox_node.appendChild(node);

        node=Createnode.createElement('ymax');
        node.appendChild(Createnode.createTextNode(sprintf('%s',num2str(str{6}))));
        bndbox_node.appendChild(node);

       lastname=str{1};
        end
        if feof(fidin)
            tempname=lastname;
            tempname=strrep(tempname,'.jpg','.xml');
            xmlwrite(tempname,Createnode);
        end
end
fclose(fidin);

file=dir(pwd);
for i=1:length(file)
   if length(file(i).name)>=4 && strcmp(file(i).name(end-3:end),'.xml')
    fold=fopen(file(i).name,'r');
    fnew=fopen([xmlpath_new file(i).name],'w');
    line=1;
    while ~feof(fold)
        tline=fgetl(fold);
        if line==1
           line=2;
           continue;
        end
        expression = '   ';
        replace=char(9);
        newStr=regexprep(tline,expression,replace);
        fprintf(fnew,'%s\n',newStr);
    end;
    fclose(fold);
    fclose(fnew);
    delete(file(i).name);
   end
end
```

### 3.xml转成4个txt,存入ImageSets

```
    %%  
    %该代码根据已生成的xml，制作VOC2007数据集中的trainval.txt;train.txt;test.txt和val.txt  
    %trainval占总数据集的50%，test占总数据集的50%；train占trainval的50%，val占trainval的50%；  
    %上面所占百分比可根据自己的数据集修改，如果数据集比较少，test和val可少一些  
    %%  
    %注意修改下面四个值  
    xmlfilepath='Annotations';  
    txtsavepath='ImageSets/Main/';  
    trainval_percent=0.9;%trainval占整个数据集的百分比，剩下部分就是test所占百分比  
    train_percent=0.7;%train占trainval的百分比，剩下部分就是val所占百分比   
    %%  
    xmlfile=dir(xmlfilepath);  
    numOfxml=length(xmlfile)-2;%减去.和..  总的数据集大小  
    trainval=sort(randperm(numOfxml,floor(numOfxml*trainval_percent)));  
    test=sort(setdiff(1:numOfxml,trainval));       
    trainvalsize=length(trainval);%trainval的大小  
  train=sort(trainval(randperm(trainvalsize,floor(trainvalsize*train_percent))));  
    val=sort(setdiff(trainval,train));   
    ftrainval=fopen([txtsavepath 'trainval.txt'],'w');  
    ftest=fopen([txtsavepath 'test.txt'],'w');  
    ftrain=fopen([txtsavepath 'train.txt'],'w');  
    fval=fopen([txtsavepath 'val.txt'],'w');  
    for i=1:numOfxml  
        if ismember(i,trainval)  
            fprintf(ftrainval,'%s\n',xmlfile(i+2).name(1:end-4));  
            if ismember(i,train)  
                fprintf(ftrain,'%s\n',xmlfile(i+2).name(1:end-4));  
            else  
                fprintf(fval,'%s\n',xmlfile(i+2).name(1:end-4));  
            end  
        else  
            fprintf(ftest,'%s\n',xmlfile(i+2).name(1:end-4));  
        end  
    end  
    fclose(ftrainval);  
    fclose(ftrain);  
    fclose(fval);  
    fclose(ftest);  
```

