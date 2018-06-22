## SSD

[SSD: Single Shot MultiBox Detector. CVPR2014.](https://arxiv.org/abs/1512.02325)

#### key idea

SSD是faster R-CNN的延续. 在faster R-CNN中, 用一个RPN网络去得到一些region proposal, 然后将这些region proposal放入到detection网络(用的是fast R-CNN中的网络, 包含ROI pooling layer)中去进行识别. faster R-CNN还是走的先生成region proposal然后再做detection的路子. SSD把两者融合在一起, 直接一个前向就得到检测结果.

faster R-CNN中有两个box regression, 一个是在RPN网络中, 用来修正预先定义的anchor得到region proposal, 一个是detection网络中, 用来修正region proposal得到最终的bounding box. SSD把这两个合二为一, 直接学习anchor的修正得到最终的detection结果.

faster R-CNN用了两个classification model, 一个是PRN网络中去判断一个anchor的objectness, 一个是detection网络中对region proposal进行分类. SSD也把这两个合二为一, 直接对anchor进行分类. 实际上, 如果一个anchor不是一个object, 它也可以被分类为background.
