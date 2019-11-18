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
首先这个文章关注的是real-time的实例分割任务，所以主要是针对Mask-RCNN two stage并且还要在进行ROI Pool/Align在进行分割导致速度过慢的问题，本文提出了一种并行的one-stage的方法。在COCO数据集上（550\*550）使用一张卡达到了33fps并且保持了29.8的mAP

# Method
文章的方法很简单，将实例分割任务分为两支，一支是预测prototypes，第二支是预测cofficients，然后通过矩阵乘进行一个线性变换得到最后的segmentation结果。（这里看不懂不要紧，往下看哈）

#### 1. Prototype Generation
prototype的生成很简单，就是在一个backbone network的输出后面又加了几个卷积层（如果要改变size的话还需要upsampleing），然后最后一层的channel数等于需要的prototype的数量  
本文用的网络结构是FCN，因为这个网络最终的feature map不仅大而且深

![](/img/yolact/prototype.png)

#### 2. Mask Coefficients
这一步顾名思义就是为每一个prototype生成一个系数，这一步就和之前的one stage detection任务一致了（本文也确实用了backbone detector来得到每隔anchor的位置，每个类别置信度和prototype系数），区别就是对每一个anchor回归（4+n+k）个系数，即4个坐标系数，n个label系数（SSD的话是n+1因为背景也算一类），k个prototype个数  
本文在这里用了tanh的激活函数，希望给coefficient一个界限，不要乱跑，因为接下来要对k个prototype图进行加加减减

![](/img/yolact/coefficient.png)
#### 3. Mask Assembly
拿一张input一张图片举例子，通过protonet得到h\*w\*k的prototype map，然后得到其中一个anchor的系数矩阵k\*1（该anchor属于的类别已经得到，这里只是用来生成segmentation map），所以直接拿prototype map和系数矩阵相乘即可得到h\*w\*1的最终结果,文章里还加了个sigmoid激活函数，来把最终结果归到0-1之间

这里在解释一下为什么可以对生成的prototype加加减减得到instance segmentation map呢，看下面这个图，这是作者文章中的展示了一个图片生成的6个prototype图，就拿最后那个两个罐子图举例子，脑补一下确实加加减减可以得到结果(第一行第一列就是第一个罐子，第一行第三列减第一列得到第二个罐子)

![](/img/yolact/interpret.png)

#### 4. Loss
loss function上面就很平凡，用了分类的loss，回归框位置的loss以及最终segmentation map的bce loss，没有对prototype map加loss因为这个prototype是instance variant的

#### 5. Pipeline
最终pipeline如下图，除去中建一些小trick，别的应该很清晰了
![](/img/yolact/pipeline.png)

#### 6. Fast NMS
传统的NMS算法大概只有5fps的速度，所以太慢了（具体的没看懂，留下了弱者的泪水TAT）

# Result
首先是和SOTA Mask-RCNN在coco上面的对比，可以看到虽然掉点了但是确实快很多，作者还指出一个现象就是IOU要求越高，他们的方法的表现就和Mask-RCNN越接近，甚至在IOU>0.95的情况的时候，还超过了SOTA，因此说明了mask质量很高
![](/img/yolact/result1.png)

本文还进行了ablation分析，可以看到fast nms基本上不掉点，同时也选出了比较好的prototype参数
![](/img/yolact/result2.png)

# Contribution
作者自述本文最大的贡献就是又快又好(fast, high-quality, general)以及提出了fast nms然后更快了

# Thought
个人觉得这个文章还是很有意思的，可能是我之前没看过prototype类型的文章，给我一种像是对图片进行各种各样的分割结果然后重新组合在一起得到一个想要的instance segmentation map

文章里还有个观点，说conv善于producing spatially coherent masks，而fc善于producing semantic vectors，所以用conv来生成prototype，用fc来生成coefficient，这也一定程度上解释了该方法生成的mask质量比较高的原因。

但我疑惑的地方是YOLO是用fc来生成坐标的（YOLO也是本文采用的backbone detector），而SSD已经把fc改成了conv，所以是否意味着如果用SSD代替YOLO，产生的coefficient会效果变差？
