## Faster R-CNN
[Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks. NIPS2015.](https://arxiv.org/abs/1506.01497)

#### key idea

这篇文章延续R-CNN系列模型. 主要解决的是怎么得到region proposal的问题. 之前的fast R-CNN是通过selective search得到约2k个region proposal, 这种获取proposal的方法在实际应用中已经成为性能瓶颈, 这篇文章提出了RPN网络(Region Proposal Network)来取代之前的selective search. RPN网络与classification网络共用前面的卷积层, 只是在共享卷积层之后加了一些特殊的结构, 所以大大降低了计算量.

* Region Proposal Networks

  输入为一幅图, 输出为图中固定的区域的bounding box修正值, 以及objectness score.

  To generate region proposals, we slide a small network over the convolutional feature map output by the last shared convolutional layer. This small network takes as input an n × n spatial window of the input convolutional feature map. Each sliding window is mapped to a lower-dimensional feature. This feature is fed into two sibling fully-connected layers—a box-regression layer (reg) and a box-classiﬁcation layer (cls).

  由于是在feature map上用滑动窗进行遍历, feature map上的滑动窗覆盖的区域对应输入图像的 receptive field, 所以实际上是输入图像每一个receptive field对应一些box regression结果和对应的objectness score.

  这里采用FCN模型, 窗口滑动对应nxn的卷积, 全连接对应1x1的卷积.

* Anchors

  前面说过, RPN网络相当于在输入图像上用一个大的滑动窗(receptive field)和一个大的步长进行滑动, 得到一些box regression输出和box classification输出. 注意这里的box regression输出是相对于输入图像滑动窗的位置而言的, 实际操作与原版R-CNN中的bounding box regression一致. 而box classification输出的为这个位置的objectness score. 这里我们对于每一个receptive field定义多个anchor, 因此每一个位置可以predict多个bounding box和objectness score. 当然, 这里predict出来的bounding box是相对于anchor而言的. (这里的box regression仅仅是RPN网络中对anchor的修正, 在随后的detection模型中, 会有一个box regression对这个修正之后的bounding box再进行修正)

  An anchor is centered at the sliding window in question, and is associated with a scale and aspect ratio. By default we use 3 scales and 3 aspect ratios, yielding k = 9 anchors at each sliding position.

  这里的Anchor具有平移不变性, 并且可以在一个位置定义不同大小不同形状的anchor (所以也可以处理不同的scale和aspect ratio), 这个是这篇文章最大的创新.

* loss

  给定一幅已标注的图像, 如何得到每一个anchor对应的box regression的真值及其objectness?  如果一个anchor与某一个ground-truth box的IoU大于0.7, 则认为它是一个object; 如果IoU小于0.3, 则认为其不是, 介于0.3和0.7之间的被忽略 (不参与loss计算). 为了照顾到少数特殊的ground-truth box, 把与某个ground-truth box具有最大的IoU的anchor也作为正样本.

  For training RPNs, we assign a binary class label (of being an object or not) to each anchor. We assign a positive label to two kinds of anchors: (i) the anchor/anchors with the highest Intersection-over-Union (IoU) overlap with a ground-truth box, or (ii) ananchor that has an IoU overlap higher than 0.7 withany ground-truth box. Note that a single ground-truthbox may assign positive labels to multiple anchors.

  classification loss就是两类的softmax loss (其实也相当于logistic loss), regression loss用的是smooth L1 loss.

* training

  这里每一次训练一个sample, 即mini-batch=1, 从中随机选出128个正的anchor和128个负的anchor进行训练, 所以训练过程很慢.

  It is possible to optimize for the loss functions of all anchors, but this will bias towards negative samples as they are dominate. Instead, we randomly sample 256 anchors in an image to compute the loss function of a mini-batch, where the sampled positive and negative anchors have a ratio of up to 1:1. If there are fewer than 128 positive samples in an image, we pad the mini-batch with negative ones.

* faster R-CNN的测试流程

  * 将输入图像过一遍公共网络, 得到feature map;
  * 用RPN网络对feature map进行处理, 得到每一个anchor的修正值(box regression结果)及其objectness score,  取objectness score大于某个阈值的anchor, 通过修正后得到一些region proposal, 这也是RPN网络的结果.
  * 将RPN网络输出的region proposal输入到detection network中(与RPN网络共用前面的一些卷积层), 得到这些proposal的修正值及其属于某一类的score;
  * 做class-wise的非极大值抑制, 得到最终的识别结果.
