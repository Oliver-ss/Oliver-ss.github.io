---
layout:     post
title:      Literature Review
subtitle:   Domain Adaptation for Sementic Segmentation
date:       2020-04-08
author:     Song
header-img: img/post_bkg_avocado3.jpg
catalog: true
tags:
    - Deep Learning
    - Domain Adaptation
---

# Introduction

>Domain adaptation is a particular case of transfer learning that utilizes labeled data in one or more relevant **source domains** to execute new tasks in a **target domain**.<br>Cited from "Deep Visual Domain Adaptation: A Survey"  [pdf](https://arxiv.org/abs/1802.03601)

Domain Adaptation在分类问题上现在已经有了不少的paper，详情可看上面提到的那个survey，但是domain adaptation在分割问题上研究还不多，并且分割问题和分类问题又存在的很大的不同，前者是pixel-wise的而后者不是，下面就是对最近几年的关于分割问题的domain adaptation的paper整理。

# Reading List

### 1.Revisiting Batch Normalization For Practical Domain Adaptation [pdf](https://arxiv.org/abs/1603.04779)
+ **Method：**这篇文章可以说是提升效果最明显最简单的domain adaptation的文章了，并且不受task的限制，只要是网络中存在BN，即可使用这个方法在target domain上重新计算新的BN层的mean和var从而达到domain adaptation的效果。

当然这个文章能取得这么好的效果主要靠的是BN层的对于层间数据规范化的能力，因为BN会对层间数据做一个channel-wise的归一化，然后再通过可学习的缩放和平移系数，使得每个层得到的输入都尽可能的服从相同的分布（假设都是高斯分布），这从一定程度上减小了对于数据集的依赖，也就减少了domain shift对网络的影响。但这里只用到了均值，标准差这两个统计数据，即使两组数据同均值，同标准差，他们并不一定同分布，所以有一些方法也会由此引入协方差矩阵，希望通过协方差矩阵来进一步规范数据（但引入协方差矩阵同样会面临新的问题比如计算量过大，本身NN的数据维度就很高，与之对应的协方差矩阵的又会是维度的平方大小），从而得到generalization的效果。个人觉得这其实是一个trade-off，即网络得到的数据var越低，那么网络的泛化能力就会提升，但是网络的业务能力也会随之下降。

### 2. Learning to Adapt Structured Output Space for Semantic Segmentation(2018CVPR) [pdf](https://arxiv.org/abs/1802.10349)
##### Assumption
文章的假设是，在语义分割的问题上，虽然不同数据集的图片会存在domain shift，比如真实图片和虚拟图片的差距，但语义分割的结果往往会具有相同性，例如排布，形状等。（论文原话：For instance, even if images from two domains are very different in appearance, their segmentation outputs share a significant amount of similarities, e.g., spatial layout and local context）
##### Method
文章主要提出了因为语义分割结果具有相似性，所以想要在output space上面构建一个相似性损失函数，但是如果直接用一些类似于SSIM的指标来构建损失函数，又无法表达这个复杂的相似性，所以很自然的借鉴了GAN的思想使用了一个dsicriminator，即利用网络来判断两个语义分割的结果是否属于同一个domain。
![](/img/literature-review/adaseg.png)
本文的核心观点就是这个output space上面的discriminator，但是为了实现SOTA，文章在featrue space上面也加了discriminator，通过这样一个加强的multi-level discriminator的模型，达到了SOTA的效果。

##### Result  
为了方便比较，只贴出GTA5->Cityscapes和SYNTHIA->Cityscapes这两组实验DA的结果
![](/img/literature-review/adaseg-1.png)
![](/img/literature-review/adaseg-2.png)


### 3. Conditional Generative Adversarial Network for Structured Domain Adaptation(2018CVPR) [pdf](http://openaccess.thecvf.com/content_cvpr_2018/papers/Hong_Conditional_Generative_Adversarial_CVPR_2018_paper.pdf)

### 4. FCNs in the Wild: Pixel-level Adversarial and Constraint-based Adaptation(2016CVPR) [pdf](https://arxiv.org/pdf/1612.02649.pdf)
这应该是第一篇做pixel-level的doamin adaptation的文章（有一说一这个文章写的是真的烂，前言不搭后语的，提出的第二个loss也没有公式，写的乱七八糟，github上面也没有实现的代码，醉了）
##### Assumption
文章把存在的domain shift分成了两类global和category shifts，一个global是由于数据集整体的变化造成的特征空间的边缘概率的不同，这个domain shift在两个比较不同的数据集中比较明显，比如生成数据集和真实数据集；第二个category是每一个种类内部的参数变化造成的doamin shift，例如不同数据集同一类的东西的大小，外观，数量都会存在不同。  
作者在这篇文章里假设不同的domain拥有相同的label space

##### Method
为了解决文章中提到的两个domain shift，作者用了两个loss来解决。对于第一个global domain shift，作者在最终输出层的前一层后面加了一个discriminator做了一个adversarial loss；对于第二个category domain shift，作者提出了一个针对每一个种类该有的size的loss，按照我的理解就是，在source domain里面，因为我们有label，所以可以统计每一类的大小，即每一类出现在每张图片里面占据图片的比例，举个例子，车载的那种道路分割图片(KITTI)，即使真实数据集和生成数据集存在shift，但是车载摄像头拍的道路图片的主旋律就是道路占据图片比较大的空间，所以如果我们的模型在target domain上预测的道路部分只占了当前图片很小的空间，那么很大概率是这个预测存在问题，所以作者要根据在source doamin上每一类大小的统计数据，给他们在target domain上预测的实际size一个无监督的loss。

至于这个loss怎么实现的，怎么梯度更新的，文章没说，代码也没有，所以我盲猜是个0-1 loss吧，即预测的东西的大小如果符合统计范围内，就没有loss，如果不符合，就loss是1。然后文章提到因为一张图片里面的很多类大家有大有小，比方说车载图片，道路就是占地很大，行人可能就是占地很小，所以对于面积比较大的那类，梯度更新时会给一个0.1的scalar平均一下。（所以如果是0-1loss的话，面积大小我觉得没关系吧，不应该看大家的出现次数么，出现次数多的种类给一个小的scalar？不懂，总之这个category loss充满了迷幻现实主义色彩，大家看看就好，也可能是我英语阅读理解太抠脚，无法理解作者的深意，欢迎大家看原文然后深度解读一下）

文章还提到说因为这个category要比较target image预测的物体的size对不对，所以不太能在模型一开始训练的时候就把这个loss加进去，因为一开始模型一通乱预测，这个loss就会很大会把模型带歪，所以先在source domain上训好了在用这个无监督loss finetune的实际上是。这个观点我还是比较赞同的，无监督loss无论是进场过早还是系数过大，都会把模型带进沟里。

![](/img/literature-review/fcns.png)
模型结构配图一张，结构上还是比较简单的。
 
##### Results
（这个文章看的过于影响心情，随便贴两个结果了）
![](/img/literature-review/fcns-1.png)

