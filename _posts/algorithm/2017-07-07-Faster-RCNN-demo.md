---
layout: post
title:  "Faster-Rcnn demo.py解析"
category: [algorithm ]
tags: [object detect,algorithms]
description: object detect
header-img: "img/pages/template.jpg"
---


1. 源代码注解  

对代码中比较重要的地方添加注释，包括自己的理解和一些参考。  

2.相关知识点补充  

补充IoU, 非极大值抑制， python的argparse模块等相关知识点。  
```
import _init_paths
from fast_rcnn.config import cfg
#im_detect 生成RPN候选框
from fast_rcnn.test import im_detect
#nms 进行非极大值抑制
from fast_rcnn.nms_wrapper import nms
from utils.timer import Timer
import matplotlib.pyplot as plt
import numpy as np
import scipy.io as sio
import caffe, os, sys, cv2
#引入argparse， 它python用于解析命令行参数和选项的
#标准模块，用于解析命令行参数
import argparse
```
```
CLASSES = ('__background__',
           'aeroplane', 'bicycle', 'bird', 'boat',
           'bottle', 'bus', 'car', 'cat', 'chair',
           'cow', 'diningtable', 'dog', 'horse',
           'motorbike', 'person', 'pottedplant',
           'sheep', 'sofa', 'train', 'tvmonitor')


NETS = {'vgg16': ('VGG16',
                  'VGG16_faster_rcnn_final.caffemodel'),
        'zf': ('ZF',
                  'ZF_faster_rcnn_final.caffemodel')}
```

def vis_detections模块：画出测试图片的bounding boxes， 参数im为测试图片； class_name 为类别名称，在前面定义的 CLASSES 中； dets为非极大值抑制后的bbox和score的数组；thresh是最后score的阈值，高于该阈值的候选框才会被画出来。  

```
def vis_detections(im, class_name, dets, thresh=0.5):
    """Draw detected bounding boxes."""
    ##选取候选框score大于阈值的dets
    inds = np.where(dets[:, -1] >= thresh)[0]
    if len(inds) == 0:
        return
    # python-opencv 中读取图片默认保存为[w,h,channel](w,h顺序不确定)
    # 其中 channel：BGR 存储，而画图时，需要按RGB格式，因此此处作转换。
    im = im[:, :, (2, 1, 0)]
    fig, ax = plt.subplots(figsize=(12, 12))
    ax.imshow(im, aspect='equal')
    for i in inds:
        #从dets中取出 bbox, score
        bbox = dets[i, :4]
        score = dets[i, -1]
        根据起始点坐标以及w,h 画出矩形框
        ax.add_patch(
            plt.Rectangle((bbox[0], bbox[1]),
                          bbox[2] - bbox[0],
                          bbox[3] - bbox[1], fill=False,
                          edgecolor='red', linewidth=3.5)
            )
        ax.text(bbox[0], bbox[1] - 2,
                '{:s} {:.3f}'.format(class_name, score),
                bbox=dict(facecolor='blue', alpha=0.5),
                fontsize=14, color='white')

    ax.set_title(('{} detections with '
                  'p({} | box) >= {:.1f}').format(class_name, class_name,
                                                  thresh),
                  fontsize=14)
    plt.axis('off')
    plt.tight_layout()
    plt.draw()
```

def demo模块：对测试图片提取预选框，并进行非极大值抑制，然后调用def vis_detections 画矩形框。参数：net 测试时使用的网络结构；image_name:图片名称。

```
def demo(net, image_name):
    """Detect object classes in an image using pre-computed object proposals."""

    # Load the demo image
    im_file = os.path.join(cfg.DATA_DIR, 'demo', image_name)
    im = cv2.imread(im_file)

    # Detect all object classes and regress object bounds
    timer = Timer()
    timer.tic()
    scores, boxes = im_detect(net, im)
    timer.toc()
    print ('Detection took {:.3f}s for '
           '{:d} object proposals').format(timer.total_time, boxes.shape[0])

    # Visualize detections for each class
    #score 阈值，最后画出候选框时需要，>thresh才会被画出
    CONF_THRESH = 0.8
    #非极大值抑制的阈值，剔除重复候选框
    NMS_THRESH = 0.3
    #利用enumerate函数，获得CLASSES中 类别的下标cls_ind和类别名cls
    for cls_ind, cls in enumerate(CLASSES[1:]):
        cls_ind += 1 # because we skipped background
        #取出bbox ,score
        cls_boxes = boxes[:, 4*cls_ind:4*(cls_ind + 1)]
        cls_scores = scores[:, cls_ind]
        #将bbox,score 一起存入dets
        dets = np.hstack((cls_boxes,
                          cls_scores[:, np.newaxis])).astype(np.float32)
        #进行非极大值抑制，得到抑制后的 dets
        keep = nms(dets, NMS_THRESH)
        dets = dets[keep, :]
        #画框
        vis_detections(im, cls, dets, thresh=CONF_THRESH)
```


def parse_args模块：解析命令行参数，得到gpu||cpu, net等。

```
def parse_args():
    """Parse input arguments."""
    #创建解析对象
    parser = argparse.ArgumentParser(description='Faster R-CNN demo')
    #parser.add_argument 向解析对象添加关注的命令行参数
    parser.add_argument('--gpu', dest='gpu_id', help='GPU device id to use [0]',
                        default=0, type=int)
    parser.add_argument('--cpu', dest='cpu_mode',
                        help='Use CPU mode (overrides --gpu)',
                        action='store_true')
    parser.add_argument('--net', dest='demo_net', help='Network to use [vgg16]',
                        choices=NETS.keys(), default='vgg16')
    #调用parser.parse_args进行解析，返回带标注的args
    args = parser.parse_args()

    return args#
```

主函数
```
if __name__ == '__main__':
    cfg.TEST.HAS_RPN = True  # Use RPN for proposals
    #解析
    args = parse_args()
    #添加路径
    prototxt = os.path.join(cfg.MODELS_DIR, NETS[args.demo_net][0],
                            'faster_rcnn_alt_opt', 'faster_rcnn_test.pt')
    caffemodel = os.path.join(cfg.DATA_DIR, 'faster_rcnn_models',
                              NETS[args.demo_net][1])

    if not os.path.isfile(caffemodel):
        raise IOError(('{:s} not found.\nDid you run ./data/script/'
                       'fetch_faster_rcnn_models.sh?').format(caffemodel))
    #gpu or cpu
    if args.cpu_mode:
        caffe.set_mode_cpu()
    else:
        caffe.set_mode_gpu()
        caffe.set_device(args.gpu_id)
        cfg.GPU_ID = args.gpu_id
    net = caffe.Net(prototxt, caffemodel, caffe.TEST)

    print '\n\nLoaded network {:s}'.format(caffemodel)

    # Warmup on a dummy image
    im = 128 * np.ones((300, 500, 3), dtype=np.uint8
    for i in xrange(2):
        _, _= im_detect(net, im)

    #im_names = ['000456.jpg', '000542.jpg', '001150.jpg',
    #            '001763.jpg', '004545.jpg']
    im_names = ['000001.jpg','000002.jpg','000003.jpg','000004.jpg','000005.jpg','1.jpg','2.jpg','3.jpg','4.jpg','5.jpg']
    for im_name in im_names:
        print '~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~'
        print 'Demo for data/demo/{}'.format(im_name)
        demo(net, im_name)

    plt.show()
```


补充：   
参考博客：[http://www.2cto.com/kf/201412/363654.html](http://www.2cto.com/kf/201412/363654.html)
一 argparse 
简介：  
argparse是python用于解析命令行参数和选项的标准模块，用于代替已经过时的optparse模块。argparse模块的作用是用于解析命令行参数，例如python parseTest.py input.txt output.txt –user=name –port=8080。   
使用步骤：   
1：import argparse   
2：parser = argparse.ArgumentParser()   
3：parser.add_argument()   
4：parser.parse_args()   
解释：首先导入该模块；然后创建一个解析对象；然后向该对象中添加你要关注的命令行参数和选项，每一个add_argument方法对应一个你要关注的参数或选项；最后调用parse_args()方法进行解析；  

二. IoU, 非极大值抑制   
参考之前博客：   
[http://blog.csdn.net/u013129427/article/details/73162817](http://blog.csdn.net/u013129427/article/details/73162817)




