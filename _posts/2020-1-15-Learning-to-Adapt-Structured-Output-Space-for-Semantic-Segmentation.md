---
layout:     post
title:      Learning to Adapt Structured Output Space for Semantic Segmentation
subtitle:   A adversarial domain adaptation method for sementic segmentation
date:       2020-1-15
author:     Song
header-img: img/post_bkg_avocado5.jpg
catalog: true
tags:
    - Deep Learning
    - Domain Adaptation
---

# Introduction
新年第一篇吧，一篇经典的用对抗训练来做segmentation问题的domain adaptation的文章

首先介绍了分割问题的domain shift，这个是因为不同的城市数据集的物体和场景的分布不同，这个主要是由于天气和光照情况的变化。如果要对每一个城市的数据集都重新标注的话，就费时费力，所以提出了分割任务的domain adaptation问题。

domain adaptation在分类问题上已经有了一定的研究，主要的研究就是在不用的domain中寻找共同的特征。同样的方法也被搬到了分割问题上，就是对模型的特征空间进行对抗训练（特征空间指的就是backbone network提出来的特征，通常处于bottleneck的位置），但是对特征空间进行对抗训练就会存在一定问题，因为分割问题还需要一些高维的特征例如纹理，形状，外观等。所以本文提出了要在输出空间上进行adaptation，因为作者发现即使是不同的数据集，但是输出空间还是类似的，大家会有相同的空间分布和局部的纹理信息。

本文的贡献主要有两点：
- 在segmentation model后面加了个discriminator来判断结果是属于source domain还是target domain并以此做了一个adversarial loss
- 虽然反向传播会使得这个loss也能更新低层特征，但是离得太远了所以更新起来比较慢，就在不同的feature layer上加了discriminator来做一个联合的对抗训练


