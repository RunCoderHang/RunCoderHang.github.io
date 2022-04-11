---
title: YOLOv5学习
tags: 
date: 2022-04-03 23:30:00
categories: 学习总结
---


先了解一下 YOLOv3 以及 YOLOv4 的网络结构以及创新点。

## YOLOv3[^1]

![8021c75455c1e8bb7ce5fd9e608f7780.png](https://runcoderhang.github.io/thumbnails/d64e17efec78408c985ccb323e943766.png)

上图三个蓝色方框内表示Yolov3的三个基本组件：

- `CBL`：Yolov3 网络结构中的最小组件，由 `Conv` + `Bn` + `Leaky_relu` 激活函数三者组成。
- `Res unit`：借鉴 Resnet 网络中的残差结构，让网络可以构建的更深。
- `ResX`：由一个 `CBL` 和 `X` 个残差组件构成，是Yolov3中的大组件。每个Res模块前面的 `CBL` 都起到下采样的作用，因此经过5次Res模块后，得到的特征图是例如：`608->304->152->76->38->19大` 小。

其他基础操作：

- `Concat`：张量拼接，会扩充两个张量的维度，例如 26*26*256 和 26*26*512 两个张量拼接，结果是 26*26*768 。Concat 和c fg 文件中的 route 功能一样。
- `add`：张量相加，张量直接相加，不会扩充维度，例如 104*104*128 和 104*104*128 相加，结果还是 104*104*128 。add 和 cfg 文件中的 shortcut 功能一样。

## YOLOv4

![5968652141db25c6c083d19ff0fefe14.png](https://runcoderhang.github.io/thumbnails/6820274921b646f6a3af40486d21d8d4.png)

- `CBM`：Yolov4网络结构中的最小组件，由 Conv + Bn + Mish 激活函数三者组成。
- `CBL`：由 Conv + Bn + Leaky_relu 激活函数三者组成。
- `Res unit`：借鉴Resnet网络中的残差结构，让网络可以构建的更深。
- `CSPX`：借鉴 CSPNet 网络结构，由卷积层和 X 个 Res unint 模块 Concate 组成。
- `SPP`：采用 1×1，5×5，9×9，13×13 的最大池化的方式，进行多尺度融合。

将Yolov4的整体结构拆分成四大板块：

1. **输入端**：这里指的创新主要是训练时对输入端的改进，主要包括 Mosaic 数据增强、cmBN、SAT自对抗训练
2. **BackBone 主干网络**：将各种新的方式结合起来，包括：CSPDarknet53、Mish 激活函数、Dropblock
3. **Neck**：目标检测网络在 BackBone 和最后的输出层之间往往会插入一些层，比如Yolov4 中的 SPP 模块、 FPN + PAN 结构
4. **Prediction**：输出层的锚框机制和 Yolov3 相同，主要改进的是训练时的损失函数 CIOU_Loss，以及预测框筛选的 nms 变为 DIOU_nms


## YOLOv5[^2]

Yolov5 的结构和 Yolov4 很相似，还是分为四个部分：

![7a25e99d3fc4dc6efff8bf0644b7a8a4.png](https://runcoderhang.github.io/thumbnails/75ed5c0079924d12b6f0912fd3945211.png)

1. **输入端**：Mosaic数据增强、自适应锚框计算、自适应图片缩放
2. **Backbone**：Focus结构，CSP结构
3. **Neck**：FPN+PAN结构
4. **Prediction**：GIOU_Loss

### 输入端

**Mosaic数据增强**

Yolov5 的输入端采用了和 Yolov4 一样的 Mosaic 数据增强的方式。随机缩放、随机裁剪、随机排布的方式进行拼接，对于小目标的检测效果不错。

**自适应锚框计算**

在Yolo算法中，针对不同的数据集，都会有初始设定长宽的锚框。

```yaml
anchors:
  - [10,13, 16,30, 33,23]  # P3/8
  - [30,61, 62,45, 59,119]  # P4/16
  - [116,90, 156,198, 373,326]  # P5/32
```

在网络训练中，网络在初始锚框的基础上输出预测框，进而和真实框groundtruth进行比对，计算两者差距，再反向更新，迭代网络参数。

在Yolov3、Yolov4中，训练不同的数据集时，计算初始锚框的值是通过单独的程序运行的。

但Yolov5中将此功能嵌入到代码中，每次训练时，自适应的计算不同训练集中的最佳锚框值。

当然，如果觉得计算的锚框效果不是很好，也可以在代码中将自动计算锚框功能关闭。

```python
parser.add_argument('--noautoanchor', action='store_true', default=False, help='disable autoanchor check')
```

**自适应图片缩放**

在常用的目标检测算法中，不同的图片长宽都不相同，因此常用的方式是将原始图片统一缩放到一个标准尺寸，再送入检测网络中。

作者认为，在项目实际使用时，很多图片的长宽比不同，因此缩放填充后，两端的黑边大小都不同，而如果填充的比较多，则存在信息冗余，影响推理速度。

因此在Yolov5的代码中datasets.py的letterbox函数中进行了修改，对原始图像自适应的添加最少的黑边。


1. Yolov5中填充的是灰色，即（114,114,114）。
2. 训练时没有采用缩减黑边的方式，还是采用传统填充的方式，即缩放到 416*416 大小。只是在测试，使用模型推理时，才采用缩减黑边的方式，提高目标检测，推理的速度。
3. 为什么 np.mod 函数的后面用 32 ？因为 Yolov5 的网络经过 **5 次下采样** ，而2的5次方，等于32。所以至少要去掉 32 的倍数，再进行取余。

### BackBone

**Focus结构**

Focus结构，在 Yolov3&Yolov4 中并没有这个结构，其中比较关键是切片操作。

对图片进行切片操作，具体操作是在一张图片中每隔一个像素拿到一个值，类似于邻近下采样，这样就拿到了四张图片，四张图片互补，长的差不多，但是没有信息丢失，这样一来，将 W、H信息就集中到了通道空间，输入通道扩充了4倍，即拼接起来的图片相对于原先的RGB三通道模式变成了12个通道，最后将得到的新图片再经过卷积操作，最终得到了没有信息丢失情况下的二倍下采样特征图。

![c2626b55f277a9ff6e606134076de917.png](https://runcoderhang.github.io/thumbnails/f7d7e6f3d7e34ffc907feff9ed9a1ff7.png)

以上图为例，原始的 608 × 608 × 3的图像输入Focus结构，采用切片操作，先变成 304 × 304 × 12的特征图，再经过一次卷积操作，最终变成 304 × 304 × 32 的特征图。

至于为什么要进行 Focus [^3]，作者：一个Focus层可以代替3个yolov3/4的层。层数减少了，FLOPS降低了。

**CSP结构**

CSPNet 全称是 Cross Stage Paritial Network，主要从网络结构设计的角度解决推理中从计算量很大的问题。

Yolov4网络结构中，借鉴了CSPNet的设计思路，在主干网络中设计了CSP结构（CSPDarknet53网络结构），不过 Yolov4 中只有主干网络使用了CSP结构。

![e7f1f998ad4a1766bb9680a4fe1ecbcf.png](https://runcoderhang.github.io/thumbnails/90eb7a3da8da4d7791e499f37d8d29c2.png)

Yolov5中设计了两种CSP结构，以 Yolov5s 网络为例，

- `CSP1_X` 结构应用于 **Backbone** 主干网络，
- 另一种 `CSP2_X` 结构则应用于 **Neck** 中。

优点：
- 增强CNN的学习能力，使得在轻量化的同时保持准确性。
- 降低计算瓶颈
- 降低内存成本

![1d5cc907d285c9a59c63881888eb7b84.png](https://runcoderhang.github.io/thumbnails/7b9413161bcb44159c10e558d3599537.png)


**SPP模块**

![88ceddfd7354648d9bb42fbb5d7f6f03.png](https://runcoderhang.github.io/thumbnails/a5f478d2c811434ca025a26c638ae448.png)

作者在SPP模块中，使用 $k={1*1, 5*5, 9*9, 13*13}$ 的最大池化的方式，再将不同尺度的特征图进行 Concat 操作。

注意：这里最大池化采用padding操作，移动的步长为1，比如13×13的输入特征图，使用5×5大小的池化核池化，padding=2，因此池化后的特征图仍然是13×13大小。

采用 **SPP模块** 的方式，比单纯的使用 $k*k$ 最大池化的方式，更有效的增加主干特征的接收范围，显著的分离了最重要的上下文特征。

## Neck

**FPN+PAN**

**FPN** 是自顶向下的，将高层的特征信息通过上采样的方式进行传递融合，得到进行预测的特征图。

在 FPN 层的后面还添加了一个 **自底向上的特征金字塔** 。FPN 层自顶向下传达强语义特征，而特征金字塔则自底向上传达强定位特征，两两联手，从不同的主干层对不同的检测层进行参数聚合

![06b22cff010b4bef27e6b65838263faa.png](https://runcoderhang.github.io/thumbnails/19d5789e4362461585237bd7df2a53df.png)

Yolov5的Neck结构中，与 Tolov4不同的是：采用借鉴CSPnet设计的CSP2结构，加强网络特征融合的能力。

![062fbb26c3a6ba228debedcb181a7676.png](https://runcoderhang.github.io/thumbnails/afbe63a4b1af4920a310f95ca8e4a05c.png)

[^1]:https://zhuanlan.zhihu.com/p/143747206
[^2]:https://zhuanlan.zhihu.com/p/172121380
[^3]:https://www.cnblogs.com/boligongzhu/p/15508249.html


