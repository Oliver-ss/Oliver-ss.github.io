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

#### 1.Revisiting Batch Normalization For Practical Domain Adaptation [pdf](https://arxiv.org/abs/1603.04779)
这篇文章可以说是提升效果最明显最简单的domain adaptation的文章了，并且不受task的限制，只要是网络中存在BN，即可使用这个方法在target domain上重新计算新的BN层的mean和var从而达到domain adaptation的效果。

当然这个文章能取得这么好的效果主要靠的是BN层的对于层间数据规范化的能力，因为BN会对层间数据做一个channel-wise的归一化，然后再通过可学习的缩放和平移系数，使得每个层得到的输入都尽可能的服从相同的分布（假设都是高斯分布），这从一定程度上减小了对于数据集的依赖，也就减少了domain shift对网络的影响。但这里只用到了均值，标准差这两个统计数据，即使两组数据同均值，同标准差，他们并不一定同分布，所以有一些方法也会由此引入协方差矩阵，希望通过协方差矩阵来进一步规范数据（但引入协方差矩阵同样会面临新的问题比如计算量过大，本身NN的数据维度就很高，与之对应的协方差矩阵的又会是维度的平方大小），从而得到generalization的效果。个人觉得这其实是一个trade-off，即网络得到的数据var越低，那么网络的泛化能力就会提升，但是网络的业务能力也会随之下降。

#### 2. Learning to Adapt Structured Output Space for Semantic Segmentation [pdf](https://arxiv.org/abs/1802.10349)
+ Assumption: 文章的假设是，在语义分割的问题上，虽然不同数据集的图片会存在domain shift，比如真实图片和虚拟图片的差距，但语义分割的结果往往会具有相同性，例如排布，形状等。（论文原话：For instance, even if images from two domains are very different in appearance, their segmentation outputs share a significant amount of similarities, e.g., spatial layout and local context）
+ Method：文章主要提出了因为语义分割结果具有相似性，所以想要在output space上面构建一个相似性损失函数，但是如果直接用一些类似于SSIM的指标来构建损失函数，又无法表达这个复杂的相似性，所以很自然的借鉴了GAN的思想使用了一个dsicriminator，即利用网络来判断两个语义分割的结果是否属于同一个domain。
![](/img/literature-review/adaseg.png)
本文的核心观点就是这个output space上面的discriminator，但是为了实现SOTA，文章在featrue space上面也加了discriminator，通过这样一个加强的multi-level discriminator的模型，达到了SOTA的效果。

+ Result(为了方便比较，只贴出GTA5->Cityscapes和SYNTHIA->Cityscapes这两组实验DA的结果):
![](/img/literature-review/adaseg-1.png)
![](/img/literature-review/adaseg-2.png)


#### 3. Conditional Generative Adversarial Network for Structured Domain Adaptation [pdf](http://openaccess.thecvf.com/content_cvpr_2018/papers/Hong_Conditional_Generative_Adversarial_CVPR_2018_paper.pdf)




