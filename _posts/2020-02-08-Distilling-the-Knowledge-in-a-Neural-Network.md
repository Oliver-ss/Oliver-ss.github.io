---
layout:     post
title:      Distilling the Knowledge in a Neural Network
subtitle:   Distilling the Knowledge in a Neural Network
date:       2020-02-08
author:     Song
header-img: img/post_bkg_avocado5.jpg
catalog: true
tags:
    - Deep Learning
    - Distilling
---

# Introduction
核心的观点就是训练的时候我们希望模型越大能力越强越好，因为大模型可以很轻易的从数据集中获取到想要的信息，并且我们还会用一些方法来把多个训练的模型集成在一起进行预测，这样也能够
使得预测的结果更加精确，但是在部署的时候，我们却因为计算能力的限制希望模型越小越好。因此本文提出了模型蒸馏，意思是把训练好的复杂的大模型迁移到部署的小模型上面。
核心方法就是使用训练模型的soft label来作为GT训练小模型，而不是数据集的hard label。

文章里面有个很有意思的观点，就是说soft label中被我们忽略的概率很小的那一类往往蕴含了一些generalize的信息（这也是符合信息熵的定义的）。举个例子，在MNIST
数据集上，有一种2的版本被判断为3的soft label是1e-6，被判断为7的是1e-9，这其实可以说明这种2比起7来更像3。

# Method
最主要的就是提出了一种带有Temperature的新softmax表达式
![](/img/distill/softmax.png)
T就是Temperature，这个一个可控的变量，当T=1的时候，这就是标准的softmax，T越高，得到的不同种类之间的概率分布就更加soft（原文：Using a higher value for T produces a softer
probability distribution over classes.）
