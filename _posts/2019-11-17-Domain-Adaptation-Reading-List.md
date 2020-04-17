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

### 1.Revisiting Batch Normalization For Practical Domain Adaptation（2016arXiv） [pdf](https://arxiv.org/abs/1603.04779)
##### Method
这篇文章可以说是提升效果最明显最简单的domain adaptation的文章了，并且不受task的限制，只要是网络中存在BN，即可使用这个方法在target domain上重新计算新的BN层的mean和var从而达到domain adaptation的效果。

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
##### Assumption
这个文章的主要的观点是说如果一定要给两个不同的domain找一个common feature space太生硬了，所以需要给这个feature space一点弹性，所以提出了使用一个generator来学习两个不同domain的feature space的不同。
##### Method
文章的采用的结构如下
![](/img/literature-review/cga.png)
主要就是给source domain生成的feature map加了一个generator生成的residual feature map，两两相加之后送给discriminator和target domain的feature map做一个adversarial loss，来使得两个domain的feature map分布相近。然后这里提到之所以generator的输入是一个noise map叠加一个low-level feature map是因为noise map用来提供随机性，然后low-level feature map是用来提供一些底层的细节信息例如边界条纹等（一般来说CNN的前面几层都是在寻找一些边界信息），因为两两组合，就能生成较为合理的residual map。

文章还专门提到，在实际训练这个模型的时候，我们把模型分为encoder，decoder，discriminator，generator四部分，那么一开始先用分割问题的cross entropy loss和实际上训练discriminator的loss来更新decoder和discriminator，然后固定住他们，再用adversarial loss来更新encoder和generator。并且考虑到GAN的不稳定性，所以实际上训练的时候，对于source domain的原始feature map和新生成的feature map都做segmentation loss。
##### Results
![](/img/literature-review/cga-1.png)
![](/img/literature-review/cga-2.png)
可以看到这个方法涨点十分明显，可以说几乎在原来结果的基础上涨了50%的mIoU，最后的adaptation之后的模型在cityscapes上面的表现都超过了40%，但这里没有提供如果我们直接用FCN-8S这个模型在cityspaces上面直接用label训练的结果，也就是如果有label，那这个模型的极限能力在哪里，所以我查了一下cityspaces的benchmark，看到FCN-8s的表现是67.1%mean IoU，所以涨点到40看起来还是比较合理的，而且FCN-8s这个模型也没有BN层，所以也不存在是否使用了AdaBN辅助的问题。下图是FCN-8s模型在cityspaces上面的表现
![](/img/literature-review/fcn-8s-baseline.png)


### 4. FCNs in the Wild: Pixel-level Adversarial and Constraint-based Adaptation(2016arXiv) [pdf](https://arxiv.org/pdf/1612.02649.pdf)
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

### 5.Curriculum Domain Adaptation for Semantic Segmentation of Urban Scenes(2017ICCV)[pdf](https://arxiv.org/abs/1707.09465)
这是找到的时间顺序上第二篇做DA for Segmentation的文章，文章的虽然有点老，但是其中的观点还是有点意思，相比于FCNS，文章不同或者说贡献主要是手动设计了两个loss function来增强model的generalize能力。
##### Assumption
文章的主要假设是觉得FCNS的方法是寻找一个common feature space，所以默认了不同的domains拥有同样的decision functions，这个假设存在问题，作者认为适应到target domain上，需要使得模型更加适应target domain的一些特性，基于这个假设，作者构建了两个target domain property的loss。
##### Method
作者主要提出了两个target domain properties，一个是Global label distributions of images，另一个是Local label distributions of landmark superpixels。所以整个模型的思路很简单，就是一个segmentation的cross entropy loss加上两个properties的各自的loss来联合训练模型。
![](/img/literature-review/cda.png)
下面简单讲一个两个target domain properties：
+ Global label distributions of images
这个其实就是在FCNS那个size constraint loss上面的延伸，把每个图片中各个类别的占比用一个分布来表示，比方说，输入一张车载图片，其中天空占0.6，马路占0.3，行人占0.1，因此构成了一个[0.6,0.3,0.1]的vector，把所有的图片都统计一遍，就有了一个distribution，所以说target domain的预测结果，要符合target doamin的这个分布，所以问题来了，我们其实没有target domain的标注，所以没有label就只能自己造pseudo label，所以作者这里就用一个backbone的Inception-Resnet V2的model来提出特征，然后用传统ML算法logistic regression，Nearest Neighbour等在source doamin上面训练，然后在target doamin上直接回归单张图片的各类占比分布（这里就是假设说这个任务相对比较简单，所以受domain shift的影响比较小，但我这里这里可以做个实验验证一下，虽然intuitively来说是正确的）
+ Local label distributions of landmark superpixels
第一个global label distribution缺少了对于位置信息的监督，所以第二个这个local label引入了位置信息，主要就是首先把图片分割成100个superpixels，然后通过在别的语义分割数据集上预训练的模型FCN来对于每一个superpixel块提取特征，然后再次根据source domain的label训练一个SVM用于预测每一个superpixel块的分布
+ Loss
无论是global distribution还是local distribution，作者都通过生成的pseudo label和实际segmentation model生成的的结果算出来的分布，构建了一个entory+KL divergence的loss：
![](/img/literature-review/cda-loss1.png)

##### Results
主要在和FCNS进行对比，显示结果远超了FCNS，达到了当时的SOTA，并且说明了和FCNS使用了同样的model，但是baseline就远超FCNS（这个也make sense毕竟FCNS这个文章无比的随意）
![](/img/literature-review/cda-1.png)
![](/img/literature-review/cda-2.png)
通过这个表可以发现引入了位置信息的SP loss涨点比较明显。个人猜想整体上的distribution来做loss，对于梯度更新来说比较模糊，所以可能效果不明显。

### 6.ADVENT: Adversarial Entropy Minimization for Domain Adaptation in Semantic Segmentation（2019CVPR）[pdf](https://arxiv.org/abs/1811.12833)
这个文章主要是在Learning to Adapt Structured Output Space for Semantic Segmentation这个文章的基础上进一步拓展了，加入了entropy map的概念以及加了个unsupervised loss，然后涨了一些点，成为了SOTA。
##### Assumption
文章的主要的假设和上面那篇文章一样，也就是不同domain的output还是存在一定的相似性的，所以可以用一个discriminator来拉进他们的距离。同时文章创新的地方就是引入了entropy的概念吧，segmentation map其实是对每一个像素点都做一个分类，softmax之后会得到了一个看起来很像概率的东西（虽然他们本质上并不是），如果我们认为这个就是概率，那么就可以通过一个公式（h,w代表map的高和宽，c代表有多少类）
![](/img/literature-review/advent-1.png)

所以如果结果十分确定，举个极端例子，在某一个点上面，得到的结果是[1,0,0,0,0]，意思是该点有100%的可能性是属于第一类，所以entropy算出来就是0，其实这个entropy就是看模型对于预测的结果是否自信。文章里面贴了个图，可以看到这个entropy map和segmentation map的区别就是仿佛entropy map在勾勒一个边界，也就是说，模型其实在边界处是没那么自信的，不知道边界到底该属于哪一类，所以entropy就会高一点。
![](/img/literature-review/advent-2.png)

##### Method
主要方法就是把模型输出的结果从一个segmentation map先转换成一个entropy map，然后用一个discriminator来对不同domain的entropy map做一个adversarial loss，然后辅助加上一个entropy minimization loss，这个loss的公式如下
![](/img/literature-review/advent-3.png)
这个loss作者说还和self-training的概念有点相近，其实本质上在利用模型生成的pseudo label来进一步对模型进行训练，只不过相比于设置一些threshold来选取一些high-confidence pixel作为label，这个entropy的方法就类似于一个soft label。感觉就是稍微用了几句话简单了证明了一下他的观点可能是有理论依据的？反正听起来还是有点道理的。
具体的模型结构就是如下
![](/img/literature-review/advent-4.png)
这里需要提一下的是，这个adversarial loss和cyclegan的不同，这里这个loss只是使得target domain的结果和source domain的接近，所以只针对target domain而不会去改变source domain的分布。
文章中没有提到具体的训练过程，只提到这个adversarial loss和entropy minimization loss的权重都是0.001，反正就是这种无监督的loss如果权重过大就很容易把训练带崩，所以我猜测这个模型也是用了在source domain上面训练好的模型来初始化。
##### Results
这个文章主要是在Adapt Seg上面进行了延伸，所以也主要只和Adapt Seg进行了对比，可以看到基本上是涨点了吧，虽然可能没有那么明显。
![](/img/literature-review/advent-5.png)
