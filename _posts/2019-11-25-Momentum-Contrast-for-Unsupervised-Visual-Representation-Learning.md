---
layout:     post
title:      MoCo
subtitle:   Momentum Contrast for Unsupervised Visual Representation Learning
date:       2019-11-25
author:     Song
header-img: img/post_bkg_avocado5.jpg
catalog: true
tags:
    - Deep Learning
    - Representation Learning
---

# Introduction
学术追星，kaiming亲自挂一作的文章。

先介绍一下无监督的特征学习，这个是NLP中比较成功的概念了，但是视觉领域因为图像的维度太高等问题还有待发展。
这篇文章讲的特征学习的方式就是利用无监督的方法来学习提取特征的方式，
举个例子，使用resnet50训练的时候，得到的结果就是最后那个1000维fc前面那个128维的fc，也就是学习了一种把图片encode成 $$128\times 1$$ 向量的方式。 
现在主流的视觉上的无监督特征学习的方法采用的是contrastive loss（观点来自于LeCun的一篇文章Dimensionality Reduction by Learning an Invariant Mapping 
大概意思就是两个相似的样本，经过降维或是说特征提取之后仍然相似，反之），具体来说就是建立一个动态的字典，这字典会存有一些来自于encode数据集样本的keys，
我们需要训练一个新的encoder使其能够实现字典查询的功能，即encode一个新的样本，使其能够和字典中对应的key尽可能的相似，和别的尽可能的不同。

本文的主要观点就是通过队列（queue）的思想建立了一个动态的字典和一个缓慢更新的encoder，由此对contrastive learning中建立字典查找的方法进行了改进。
然后全面涨点，MoCo方法训练出来的无监督pretrained model在7个检测和分割的赛道上都超过的通过有监督学习学出来的模型。

# 
