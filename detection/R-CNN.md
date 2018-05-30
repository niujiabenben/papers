## R-CNN
[Rich feature hierarchies for accurate object detection and semantic segmentation.
CVPR2014.](https://arxiv.org/abs/1311.2524)

#### key idea

这是用deep learning做detection的开山之作.
这篇文章之前, deep learning在分类问题上取得了重大突破.
在检测问题上也有一些简单的尝试, 比如直接回归bounding box, 但效果不是很好.
这使得作者思考如何从detection问题上抽出classification的任务交给deep learning去做.
于是有了region proposal + feature extraction + svm classification这个框架.
第一步, 用selective search提取region proposal, 一副图像大约提2k个;
第二步, 取CNN最后一层4096维的输出作为特征;
第三步, 将这些特征送入SVM进行分类.
这个方法在PASCAL VOC 2010上的mAP为: 53.70%, 传统方法的mAP为: 35.1%.

* 这里用selective search做region proposal是为了和别人比较:

  While R-CNN is agnostic to the particular region proposal method,
  we use selective search to enable a controlled comparison with prior detection work.

* 提特征的时候, 直接将每一个region proposal缩放到指定大小, 并在四周留16个像素的padding,
  总的大小为227x227:  

  Regardless of the size or aspect ratio of the candidate region,
  we warp all pixels in a tight bounding box around it to the required size.
  Prior to warping, we dilate the tight bounding box so that at the warped size
  there are exactly p pixels of warped image context around the original box
  (we use p = 16).

* 每一个class对应一个二分类的SVM, 得到所有的region proposal的分类结果之后, 做一个非极大值抑制:

  Given all scored regions in an image, we apply a greedy non-maximum suppression
  (for each class independently) that rejects a region if it has an
  intersection-over-union (IoU) overlap with a higher scoring selected region larger than
  a learned threshold.

* CNN训练过程: 先用ImageNet进行pre-train, 再用region proposal进行fine-tune.
  正样本条件: IoU > 0.5, 其他为负样本. 训练时, 每一个mini-batch包括32个正样本和96个负样本
  (因为正样本的个数非常少):

  We treat all region proposals with ≥ 0.5 IoU overlap with a ground-truth box as
  positives for that box’s class and the rest as negatives.

  In each SGD iteration, we uniformly sample 32 positive windows (over all classes) and
  96 background windows to construct a mini-batch of size 128. We bias the sampling
  towards positive windows because they are extremely rare compared to background.

* 为什么CNN只用来体特征, 分类要另外用SVM?

  It would be cleaner to simply apply the last layer of the ﬁne-tuned network, which
  is a 21-way softmax regression classiﬁer, as the object detector. We tried this and
  found that performance on VOC 2007 dropped from 54.2% to 50.9% mAP.

* Bounding box regression步骤

  这一步和提特征的操作密切相关. 用selective search得到region proposal之后,
  将此region加padding之后进行缩放. 由于region的大小和性质等信息已经丢失,
  所以在做回归的时候需要针对这个bounding box的宽高进行归一化.

  这个步骤只能对bounding box进行微调, 因为提特征的时候对region加的padding是有限的,
  所以, 只有(大部分的)ground truth也在加了padding的region的内部, 提的特征才携带完整的ground
  truth信息, 这种情况下才可以做regression.

  The second issue is that care must be taken when selecting which training pairs
  (P, G) to use. Intuitively, if P is far from all ground-truth boxes, then the task
  of transforming P to a ground-truth box G does not make sense.

#### drawbacks

* 慢(13s/image on a GPU or 53s/image on a CPU), 每一个region proposal都要过一遍CNN,
  有许多重复计算.

* 直接把region proposal缩放成227x227, 损失了其aspect ratio的信息,
  但似乎ImageNet分类也是这样做的.

* 用CNN来提特征用SVM来分类这种选择略显多余, 直接用CNN来做detector会更简便一些.

#### question

* 为什么直接回归bounding box效果不好?

* standard hard negative mining method 具体是怎么做的?
