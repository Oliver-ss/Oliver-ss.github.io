---
layout:     post
title:      YOLACT
subtitle:   Real-time Instance Segmentation
date:       2019-11-18
author:     Song
header-img: img/post_bkg_avocado5.jpg
catalog: true
tags:
    - Deep Learning
    - Instance Segmentation
---

# Introduction
首先这个文章关注的是real-time的实例分割任务，所以主要是针对Mask-RCNN two stage并且还要在进行ROI Pool/Align在进行分割导致速度过慢的问题，本文提出了一种并行的one-stage的方法。在COCO数据集上使用一张卡打到了33fps。

# Method
文章的方法很简单，将实例分割任务分为两支，一支是预测prototypes，第二支是预测cofficients，然后通过矩阵乘进行一个线性变换得到最后的segmentation结果。（这里看不懂不要紧，往下看哈）

#### 1. Prototype Generation
prototype的生成很简单，就是在一个backbone network的输出后面又加了几个卷积层（如果要改变size的话还需要upsampleing），然后最后一层的channel数等于需要的prototype的数量  
本文用的网络结构是FCN，因为这个网络最终的feature map不仅大而且深。
<img src="img/yolact/prototype.png" width="50%" height="50%" div align=center/>

#### 2. Mask Coefficients
这一步顾名思义就是为每一个prototype生成一个系数，这一步就和之前的one stage detection任务一致了，对每一个anchor回归（4+n+k）个系数，即4个坐标系数，n个label系数（SSD的话是n+1因为背景也算一类），k个prototype个数  
本文在这里用了tanh的激活函数，希望给coefficient一个界限，不要乱跑，因为接下来要对k个prototype图进行加加减减

#### 3. Mask Assembly
$$
M=\sigma(PC^T)
$$

# Contribution

# Thought
