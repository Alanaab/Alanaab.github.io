---
layout:     post
title:      "Object detection based on SSD"
date:       2020-12-14 16:45:00
author:     "Alanby"
header-img: "img/post-seu-room02.jpg"
tags:
    - SSD
    - Object detection
    - 
---


<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$$','$$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>


# 基于SSD的目标检测模型（论文复现）



## 1. 简介

近年来，目标检测的模型不断涌现，如下图所示。不同于 Fast R-CNN 等，本文复现 的 SSD 算法是一种直接对边界框和目标类别预测的算法。与 two stage 算法相比，速度方 面有较大提升。复现着重于对整体模型的搭建分析和 Priois、Feature Map 等概念的理解， 以及如何将 VGG-16 图像分类框架修改成为目标检测框架、框架如何进行定位等。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_01.png)



## 2. 问题与难点



### 2.1 全连接层与卷积层的替换

​		由于本论文是以 VGG-16 的图像分类模型为基础，但是此目标检测并不需要原先网络 最后的全连接层，因为全连接层在原网络中是用来做分类输出的，这里只需要卷积过后产 生的特征映射矩阵。因此，这里需要对原先的网络模型修改，但是如何将全连接层转化为 卷积层呢？转化过后，它和原来的效果有什么差别，是不是能够完全等价呢？在原论文中并 没有详细描述具体这一原理，这里的摸索和理解需要不少时间。



### 2.2 先验矩形框

还有一点比较难理解的是，获取预测框需要在卷积压缩后的特征映射图中，设定一定 的框选规则，以每个像素点为中心画框，SSD 模型中选取了六个不同尺度的特征映射图来 框取先验框，分别选取的层数是四、七、八、九、十和十一层。一个疑问是为什么要选择六 层，还有一个就是对于一、三和六层不选为特征映射图的原因是什么？这点到现在也没太 明白，只能猜测是起初几层卷积出来的特征图还不够具有代表性。 对于每个特征图中的先验框，和原图大小尺寸比例也不一致，如何统一将所有先验框 映射成为原图实际的框的坐标和分类。对于上面讲到的每一个特征映射图，每个规格的图 1 课程报告 需要两个卷积层，一层用来预测位置，一层预测类别，那么合计就有十二层，实现起来也是 比较复杂。



### 2.3 矩形框的舍取

文中一共给出了六个层，以每个像素点为中心，选取 6 个框，那么每层的先验框个数 为像素点个数与六的乘积，算下来一共有 8732 个框，对于计算代价来说，如果全部框都计 算，第一点是计算量很大，程序运行时间很长；第二点是多有框中，负样本的框肯定占大多 数，这样会因此欠拟合。所以还需要考虑如何筛选先验框。



### 2.4 损失函数设计

之前说到 SSD 目标检测模型是 one stage 模型，一次性预测矩形框的位置和所框的类 别。在计算代价的时候，之前接触过的 CostFunction 都是针对带个预测的，不如说仅仅针 对框所在的位置有一个损失函数。但是文中提出在计算损失函数时，将位置的偏差代价和 类别的预测的代价相加作为整体的损失函数，看起来简单，但还是觉得挺巧妙的。



## 3. 整体思想

SSD 的整体思想是在不同卷积层的特征映射图上，以每一个像素为中心，按照一定的 规则画不同比例的矩形框，并将这些矩形框映射回到原图中，与给定的 label 进行比较，计 算出框取的位置偏差和分类的置信偏差，根据偏差更新模型参数，不断迭代，最后希望达到 预选的框和给定的框几乎重合。对于整个框架的理解，绘制出以下程序流程图和模型结构 图，并按照程序框架步骤进一步详细阐述算法的思想。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_02.png)



![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_03.png)



### 3.1 特征图选取

网络总体来说一共有三个部分：

- **基础网络：**提取低尺度的特征映射图
- **辅组网络：**提取高尺度的特征映射图
- **预测网络：**输出特征映射图的位置信息和分类信息

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_04.png)

即图像批量输入后，依次经过基础卷积层和辅组卷积层，在基础卷积层中提取出两层 低尺度的特征图，在辅组卷积层中选取其中四层高尺度特征图，得到这六层特征图后，接下 来的工作都是围绕它们进行了。



### 3.2 先验框绘制

得到六层特征图后，我们可以设置一定的规则，在这些图上绘制矩形框。在 SSD 算法 中，是以像素点为中心，绘制了大小不同的矩形框，如下图：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_05.png)

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_06.png)

这时候可能会有疑问，第一是为什么要在特征图上画，而不在原图上绘制，理论上来 说，原图一般比较大，比如这里的图为 300×300，按照相同的方法，就有 300×300×6 个预 选框了，数量远远大于 8732，计算上来说是不可取的。  

第二点疑问是在这些特征图上绘制框，能覆盖完原图上所有可能的位置吗，这里由于 是多尺度特征图，而且是从六层不同的图中遍历，情况基本都能覆盖完，也就是说原图上的 target 基本都会被某些矩形框框中。



### 3.3 预测卷积层

得到 8732 个先验证框后，由于不同的特征图大小不一，所以得到的框也大小不一，下 一步需要做的就是将所有框的位置和类别映射到同一标准，也就是映射回没卷积前的原图 中。对于每个特征图，我们需要预测：

- 边界框的位置
- 框取的类别

所以对于每个特征图，我们需要两个对应大小的卷积层，一个是局部位置预测，一个 用于类预测，其实就是相当于两个公式，一个计算坐标，一个计算类别。因此我们一共有 12 个卷积层，也就是 12 个不同的运算公式，用于计算这 8732 个矩形框的位置和类别信息。 到此为止，我们已经知道了预测的所有矩形框的信息，同时给定的正样本的矩形框我们也 已知。接下里就是将预测情况和真实情况做匹配。



### 3.4 预测匹配

#### 3.4.1 正样本选取

SSD 算法中，使用了预测框与真实框的 Jaccard 重叠小于 0.5 为负样本，否则为正样本，也就是我们常说的 IOU 区域。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_07.png)

但是我们有 8732 个预测框，真实需要的框的地方其实仅仅是图中的一部分，所以这样 下来产生的负样本数必然远远大于正样本数，如果将其全部用于模型参数更新的话，正样 本的效果基本可以忽略，造成 Underfitting。因此我们要想办法减少负样本的数量。



#### 3.4.2 负样本难例挖掘

文中提出的方法是用难例挖掘排除一部分负样本，思想就是将所有负样本按照正向传 播计算出来的 loss 排序，选择 loss 最高的前 n 个预测框与候选负样本集匹配，匹配成功则 作为最终的负样本，参与到反向传播中去，并控制最终的正负比例为 1：3 左右。



### 3.5 LOSS计算

前面提到，SSD 是同时预测位置与类别，因此它的损失函数是两者的代价相加得到的。

#### 3.5.1 位置LOSS

对于位置 loss，我们需要将前面步骤计算出来的正样本回归到 Ground truth box 中去， 这样才能计算坐标的偏差损失。坐标的损失以以下计算偏差给出，真实的矩形框坐标表示为：
$$
Box(c_x, c_y, w, h)
$$
预测的矩形框坐标假设表示为：
$$
	Box(\hat{c}_x, \hat{c}_y, \hat{w}, \hat{h})
$$
则偏差表示为：
$$
\begin{aligned}
		g_{c_x} &= \frac{c_x- \hat{c}_x}{\hat{w}}\\
		g_{c_y} &= \frac{c_y- \hat{c}_y}{\hat{h}}\\
		g_w &= log(\frac{w}{\hat{w}})\\
		g_h &= log(\frac{h}{\hat{h}})\\
		offset &= (g_{c_x},g_{c_y}, g_w, g_h)
	\end{aligned}
$$
对于位置的偏差，文中采用的是 Smooth L1 loss：
$$
L_{loc} = \frac{1}{n_{pos}}(\sum_{pos}Smooth L_1 Loss)
$$
对于为什么要使用 Smooth L1 loss，文中没有给出解释，在 Fast R-CNN 中有提及，Smooth L1 是从 L1 和 L2 变换过来的：
$$
\begin{aligned}
		L_1(x) &= |x|\\
		L_2(x) &= x^2\\
	\end{aligned}
$$

$$
Smooth_{L_1}(x)=
    \left\{
    \begin{array}{cc}
    0.5x^2  & |x| < 1 \\
    |x|-0.5 & otherwise
    \end{array}
    \right.
$$

将 Smooth L1 求导得：
$$
    \frac{dSmooth_{L_1}}{dx}=
    \left\{
    \begin{array}{cc}
    x  & |x| < 1 \\
    \pm1 & otherwise
    \end{array}
    \right.
$$
从以上公式可以看出 smooth L1 在 x 较小时，对 x 的梯度也会变小，而在 x 很大时，对 x 的梯度的绝对值达到上限 1，也不会太大以至于破坏网络参数。smooth L1 loss 在 |x|>1 的部分采用了 L1 loss，当预测值和目标值差值很大时，原先 L2 梯度里的 (f(x)-Y) 被替换 成了 ±1，这样就避免了梯度爆炸，也就是它更加健壮。而仔细求导看，L1 导数恒为 1，会 在稳定值附近波动，难收敛；L2 对导数没有上界限制，容易产生梯度爆炸。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_08.png)



#### 3.5.2 类别LOSS

对于类别得置信度损失函数，文中使用的是交叉熵损失：
$$
	L_{conf} = \frac{1}{n_{pos}}(\sum_{pos}CE Loss + \sum_{hard neg} CE Loss)
$$

$$
CEH(p,q) = E_p[-logq] = - \sum_xp(x)logq(x) = H(p) + D_{KL}(p||q)
$$

其中 q、p 为不同分布，它描述了概率分布之间的距离，当交叉熵越小说明二者之间越接近。



## 4.测试结果

实验中，一共有用到了数据集 2000 张，红绿灯各 1000 张。训练集：测试集 = 9：1。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_09.png)

在 tensorboard 中可视化训练集的 PR 曲线和 loss 曲线：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_10.png)



![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_11.png)

合并红绿灯，训练集的部分统计结果如下：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_12.png)
$$
	\begin{aligned}
		Precision &= \frac{True\, positive}{True\, positive + False\, positive} = \frac{195}{195+2} = 0.98\\
		Recall &= \frac{True \,positive}{True \,positive + False \,negative} = \frac{195}{195+5} = 0.97
		\end{aligned}
$$
但是在测试机中的表现强差人意，有不少重复框取，而且还有一部分没有框，对于有一 些干扰的图片，还有一些框错的：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_13.png)

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_14.png)

整个测试集中，mAP 仅有 53.8%，比论文中提到的 SSD 在 VOC2007 上测试的 74.3% 的 mAP 相差了百分之二十多，显然效果不太理想。



## 5. 讨论分析

实验过后，mAP 结果不是很理想，向比较了解这方面的同学请教了一下，发现了不少 问题，总结出来了以下几点可以提升的地方和可能存在的问题。

### 5.1 分析问题

#### 5.1.1 过拟合

根据测试集和原本训练集的表现差异来看，训练的模型过拟合很严重，产生的原因可 能有两点：

- 训练过程中，由于输入的训练集不大，而且迭代的次数较多，很可能拟合了一些没什 么代表性的特征。
- 还有原论文中的 SSD 是针对 VOC2007 中 20 个种类进行目标检测的，而本次输入只 有红绿灯，训练集的数量级可能与模型本身的复杂度不匹配。
- 

#### 5.1.2 重叠框

测试的结果有像下图中的红灯被重复框选的情况，这里我对预测出来的框并没有再进 一步分析。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/SSD_15.png)



#### 5.1.3 Loss问题

当训练到第三个 epoch 后，loss 开始出现了 inf 或者 nan 的情况, 分析了下原因，可能 有几下几种：

- 学习率可能调太大了，导致梯度变化剧烈，可能会出现无穷大或 nan 等情况。
- 损失函数不合适，由于不同模型出来的偏差数值大小不一致，使用不当的损失函数也 可能导致 loss 跑飞。
- 数据集本身有不稳定的特征或者是激活函数不恰当。



### 5.2 提升空间

#### 5.2.1 Data augmentation

和同学讨论分析，针对本次红绿灯检测中，较大的影响因素应该是亮度和红绿灯的显 示面积。针对上面数据集数量较少的问题，可以采取改变不同的亮度做数据增广，而且这 样在一定的层度上可以降低过拟合。



#### 5.2.2 非极大值抑制

对输出的结果有重叠框，这个问题比较好解决，按照置信度将重叠度高的框筛选掉。具体步骤为：

- 根据框的置信度得分进行排序；
- 选择得分最高的边界框添加到输出列表；
- 计算所有边界框的面积；
- 计算置信度最高的框和其他候选框的 IOU；
- 删除 IOU 大于一定阈值的候选框。



#### 5.2.3 调整学习率

针对上面 loss 出现的问题，可以尝试采取以下几个措施进行调整：

- 相应调小学习率，看是否还会出现 nan 等情况；
- 爬取更多红绿灯图片，或者更加检测类别，适应模型复杂度；
- 尝试更多的学习率调整策略，比如说 StepLR、ExponentialLR 等等。
- 可以尝试做权值迁移，用已经训练好的 VGG-16 的模型参数作为 SSD 模型的初始化 参数。



## 6. 总结

本次实验体会比较深刻的就是，对于数据处理部分耗费的时间比其他地方都要多！本 次 2000 张图片的矩形框标注是个体力活，最后叫上了另外一个同学一起合力标注了 2000 张，耗费了一个上午和一个下午的时间，紧接着是按照 VOC2007 的数据格式，生成 txt 索 引等，保守估计，本次实验但单对数据的标注、索引生成、pytorch 数据载入网络等耗时大 概 3 天，当然一方面是刚刚接触 pytorch，还不是很熟悉。对于模型参数的调整，经验不足， 比如这次 loss 出现 nan 等情况，折腾了很久，最后把学习率降低到 0.0005 才没出现 nan 的情况，这方面的提升只能通过多看多做，积累经验了。 尝试做这个的目的其实是想借助 SSD 了解一下整个目标检测的流程大概是怎样的，和 分类比起来，要多哪些步骤，在 pytorch 中，实现起来有何异同。整体下来本实验还是达到 了自己起初的目的，对图像分类和目标检测的基本套路有了一定了解。 有一点很好奇的是，那些网络的结构，或者说改进，前人是怎样想到的，好像论文中也 没有给出理论根据说网络这样搭的原因，可能是经过多次实验得出的结果。兴趣是最好的 老师，接下来想借着这个机会继续深入了解以下其他模型和其他方面，比如说 GAN、NLP 等等，另外希望疫情早点结束，回归正常的校园生活。



## 7. 附录

主要是 SSD300 模型搭建和 loss 计算代码：

```python
class SSD300(nn.Module):
    """
    The SSD300 network - including the base VGG network, auxiliary, and prediction convolutions.
    """
		def __init__(self, n_classes):
			super(SSD300, self).__init__()
			self.n_classes = n_classes
			self.base = VGGBase()
			self.aux_convs = AuxiliaryConvolutions()
			self.pred_convs = PredictionConvolutions(n_classes)
			self.rescale_factors = nn.Parameter(torch.FloatTensor(1, 512, 1, 1)) 
			nn.init.constant_(self.rescale_factors, 20)
			# Prior boxes
			self.priors_cxcy = self.create_prior_boxes()

		def forward(self, image):
			"""
			Forward propagation.
			"""
			conv4_3_feats, conv7_feats = self.base(image)  
			# Rescale conv4_3 after L2 norm
			norm = conv4_3_feats.pow(2).sum(dim=1, keepdim=True).sqrt() 
			conv4_3_feats = conv4_3_feats / norm  
			conv4_3_feats = conv4_3_feats * self.rescale_factors  
	
			conv8_2_feats, conv9_2_feats, conv10_2_feats, conv11_2_feats = \
				self.aux_convs(conv7_feats)  # (N, 512, 10, 10),  (N, 256, 5, 5), (N, 256, 3, 3), (N, 256, 1, 1)

			# Run prediction convolutions
			locs, classes_scores = self.pred_convs(conv4_3_feats, conv7_feats, conv8_2_feats, conv9_2_feats, conv10_2_feats,
												conv11_2_feats)  
			return locs, classes_scores
		

	class MultiBoxLoss(nn.Module):
    """
    The MultiBox loss, a loss function for object detection.
    """
		def __init__(self, priors_cxcy, threshold=0.5, neg_pos_ratio=3, alpha=1.):
			super(MultiBoxLoss, self).__init__()
			self.priors_cxcy = priors_cxcy
			self.priors_xy = cxcy_to_xy(priors_cxcy)
			self.threshold = threshold
			self.neg_pos_ratio = neg_pos_ratio
			self.alpha = alpha

			self.smooth_l1 = nn.L1Loss()
			self.cross_entropy = nn.CrossEntropyLoss(reduce=False)

		def forward(self, predicted_locs, predicted_scores, boxes, labels):
			"""
			Forward propagation.
			"""
			batch_size = predicted_locs.size(0)
			n_priors = self.priors_cxcy.size(0)
			n_classes = predicted_scores.size(2)

			assert n_priors == predicted_locs.size(1) == predicted_scores.size(1)

			true_locs = torch.zeros((batch_size, n_priors, 4), dtype=torch.float).to(device)  
			true_classes = torch.zeros((batch_size, n_priors), dtype=torch.long).to(device) 

			# For each image
			for i in range(batch_size):
				n_objects = boxes[i].size(0)

				overlap = find_jaccard_overlap(boxes[i],
											self.priors_xy)  # (n_objects, 8732)

				# For each prior, find the maximum overlap
				overlap_for_each_prior, object_for_each_prior = overlap.max(dim=0)  # (8732)

				_, prior_for_each_object = overlap.max(dim=1)  # (N_o)

		
				object_for_each_prior[prior_for_each_object] = torch.LongTensor(range(n_objects)).to(device)

				overlap_for_each_prior[prior_for_each_object] = 1.

				label_for_each_prior = labels[i][object_for_each_prior]  # (8732)
				label_for_each_prior[overlap_for_each_prior < self.threshold] = 0  # (8732)
				# Store
				true_classes[i] = label_for_each_prior
				# Encode center-size object coordinates
				true_locs[i] = cxcy_to_gcxgcy(xy_to_cxcy(boxes[i][object_for_each_prior]), self.priors_cxcy)  # (8732, 4)

			# Identify priors that are positive
			positive_priors = true_classes != 0  # (N, 8732)

			# LOCALIZATION LOSS
			loc_loss = self.smooth_l1(predicted_locs[positive_priors], true_locs[positive_priors]) 

			n_positives = positive_priors.sum(dim=1)  # (N)
			n_hard_negatives = self.neg_pos_ratio * n_positives  # (N)

			# First, find the loss for all priors
			conf_loss_all = self.cross_entropy(predicted_scores.view(-1, n_classes), true_classes.view(-1)) 
			conf_loss_all = conf_loss_all.view(batch_size, n_priors) 

			# We know which priors are positive
			conf_loss_pos = conf_loss_all[positive_priors]  # (sum(n_positives))
			# Next, find which priors are hard-negative
			conf_loss_neg = conf_loss_all.clone()
			conf_loss_neg, _ = conf_loss_neg.sort(dim=1, descending=True)  
			hard_negatives = hardness_ranks < n_hard_negatives.unsqueeze(1)  
			conf_loss_hard_neg = conf_loss_neg[hard_negatives]  
			# As in the paper, averaged over positive priors only, although computed over both positive and hard-negative priors
			conf_loss = (conf_loss_hard_neg.sum() + conf_loss_pos.sum()) / n_positives.sum().float() 
			# TOTAL LOSS
			return conf_loss + self.alpha * loc_loss
```

