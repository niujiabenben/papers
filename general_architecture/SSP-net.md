## SSP-net
[Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition.
ECCV2014](https://arxiv.org/abs/1406.4729)

#### key idea

这篇文章解决的是CNN模型需要固定输入大小这一问题. 总的来说, CNN模型可以分为卷积部分和全连接部分.
卷积部分实际上不需要固定大小的输入, 全连接部分才需要. 因此, 在卷积部分加一层SSP层,
在不同大小的feature map中获得固定长度的特征, 作为全连接部分的输入. 这是一个很straight
forward的idea.

* Spatial pyramid pooling

  输入为任意尺度的feature map, 输出为固定长度的特征.

  与传统的pooling固定kernel size不同, SSP不固定kernel size, 反而固定输出的feature map的大小,
  使得kernel size随着输入的feature map的大小的变化而变化. 比如: 要得到2x2的输出,
  如果输入为4x4, 那么kernel为2x2; 如果输入为8x8, 那么kernel为4x4.

  SSP固定多个输出大小(比如论文中采用4x4, 2x2, 1x1三个输出大小), 得到不同尺度的特征,
  然后将这些特征拼到一起, 形成最终的特征.

* SSP-net for detection

  这篇文章出现在R-CNN之后不久. R-CNN用selective search提取约2k个region proposal,
  对每一个region proposal都过一遍CNN抽取特征. 对于一幅图像, R-CNN需要做约2k个CNN前向操作.

  SPP-net提出, 只需要将全图过一遍CNN网络, 对每一个region proposal, 只需要取feature
  map上对应的区域即可. 这也是fast R-CNN的key idea.

#### drawbacks

* 用SSP-net做detection时, 从feature map获取region proposal的feature这一操作中,
  region proposal与feature map中对应的位置并不是完全match的, 这样会损失一些精度.
