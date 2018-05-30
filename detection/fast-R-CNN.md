## fast R-CNN
[Fast R-CNN. ICCV2015.](https://arxiv.org/abs/1504.08083)

#### key idea

本文在R-CNN方法和SPP-net方法的基础上发展而来.

**R-CNN的缺点**

* R-CNN是一个multi-stage的流程, 首先需要在提取到的region proposal上训练一个CNN,
  然后将这个CNN作为一个feature extractor, 在抽取出的feature上训练多个class-wise的二分类的SVM;
  最后还需要做一个bounding box regression. 这些步骤都是独立的, 而根据经验,
  end-to-end的方法精度会更高.

* R-CNN在训练二分类SVM的时候需要抽取所有训练样本的特征, 这样需要大量的计算和存储资源.

* R-CNN在测试的时候对所有的proposal过一遍CNN抽特征, 使得R-CNN的测试过程很慢.

**SPP-net的缺点**

SPP-net修改了R-CNN的测试过程, 先对全图过一遍CNN得到feature map, 然后在feature
map上找每一个region proposal对应的区域, 得到该proposal的特征,
注意这里的特征不是直接送到SVM中进行分类, 而是又经过了几层全连接,
得到最后一层全连接的输出送到SVM中进行分类. (R-CNN提的feature也取的是最后一层全连接).

在训练CNN的过程中, SPP-net的卷积部分只作为特征提取器, 不参与训练.
所以用SPP-net做detection似乎是把一整个的CNN拆成了卷积部分和全连接部分.

**Fast R-CNN**

fast R-CNN实际上是在SPP-net上改进而来. SPP-net在训练CNN的方法上比较ugly,
fast R-CNN统一了SPP-net的特征提取和分类, 将其归入到一个CNN中.
训练过程只需要提供一幅图及其region proposal, 也是end-to-end的.

* RoI pooling layer

  在feature map上, 将每一个proposal对应的区域划分成7x7的patch, 取每一个patch的最大值作为输出.
  其思想和SPP-net一样:
  The RoI layer is simply the special-case of the spatial pyramid pooling layer used in
  SPPnets in which there is only one pyramid level.

* loss

  每一个proposal对应两个部分的loss: classification loss确定这个proposal属于哪一类,
  bounding-box regression loss确定这个最后的bounding box与proposal的相对位置.
  相对于R-CNN来说, 前面一个loss对应binary SVM分类器, 后面一个loss对应bounding box
  regression, 作用是对proposal的位置进行微调.

* mini-batch selection

  每一个mini-batch取2个image, 然后从每一个image中取64个region proposal:

  During ﬁne-tuning, each SGD mini-batch is constructed from N = 2 images, chosen
  uniformly at random (as is common practice, we actually iterate over permutations
  of the dataset). We use mini-batches of size R = 128, sampling 64 RoIs from each image.

    
