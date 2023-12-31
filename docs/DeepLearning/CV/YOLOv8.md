---
title: 浅析YOLOv8的改进
---

## YOLO

### 非最大值抑制

**非最大值抑制（$NMS$）**是目标检测算法中使用的一种后处理技术，用于减少重叠边界框的数量并提高整体检测质量。目标检测算法通常会围绕同一对象生成多个具有不同置信度分数的边界框。NMS会过滤掉多余和无关的边界框，仅保留最准确的那些。



### YOLO: You Only Look Once

**$YOLO$**是一种目标检测算法，由Joseph Redmon等人在2016年提出。它的核心思想是将目标检测任务转化为一个回归问题，并通过单个神经网络同时进行目标的定位和分类，实现实时高效的目标检测。



$YOLO$算法的原理和核心思想如下：

- **网格划分**：首先，将输入图像划分为固定大小的网格。每个网格负责检测该网格中的物体。不同大小的图像可能会产生不同数量的网格。

- **边界框预测**：每个网格预测多个边界框（bounding boxes），每个边界框包含一个物体。边界框由一组坐标值表示，包括边界框的中心坐标、宽度和高度。

- **物体分类**：对于每个边界框，使用分类器来预测物体的类别。通常使用卷积神经网络（CNN）来提取特征，并使用全连接层进行分类。

- **置信度评估**：每个边界框还预测一个置信度分数，表示该边界框中存在物体的概率以及边界框的准确度。

- **非极大值抑制**：对于每个类别，使用非极大值抑制（Non-Maximum Suppression，$NMS$）来去除重叠边界框。选择具有最高置信度的边界框，并且剔除与其重叠度较高的边界框。

![1703336955973.png](http://pic.moguw.top/i/2023/12/23/6586dbfebc620.png)

 

​	$YOLO$ 算法的优点在于它的速度和实时性。由于 $YOLO$ 将目标检测任务转化为一个单次前向传递的回归问题，不需要候选区域（region proposals）的生成过程，因此能够实现实时的目标检测。此外，$YOLO$ 在整个图像上进行全局优化，能够捕捉到物体的全局上下文信息。

​	然而，$YOLO$ 也存在一些局限性，如难以检测小目标和密集目标，以及对目标的定位精度可能不如一些两阶段的目标检测算法。为了解决这些问题，$YOLO$ 的改进版本 $YOLOv2$、$YOLOv3$ 和 $YOLOv4$ 等相继提出，不断提升了目标检测的性能和精度。



## YOLOv8

由于 $Ultralytics$ 公司并未提供相关论文，这里援引 $MMYOLO$ 的 $YOLOv8$ 模型结构图

![1703337062676.png](http://pic.moguw.top/i/2023/12/23/6586dc6a09144.png)



$YOLOv8$ 算法参考了 $EfficientDet CNN$ 设计了“$Backbone-Neck-Head$”结构

![1703337131387.png](http://pic.moguw.top/i/2023/12/23/6586dcae94260.png)

### BackBone

在Backbone中实现了多尺度卷积特征提取，高中低三层的图像卷积克服了$YOLOv1$模型对小物体识别困难的缺陷；

![1703337179192.png](http://pic.moguw.top/i/2023/12/23/6586dcddbf0da.png)

​	相比较 $YOLOv5$，$YOLOv8$ 在Backbone中将 $C3$ 结构换成了梯度流更丰富的  $C2f$  结构，其中 $C2f$ 结构参考了 $CSPNet$ （Cross Stage Partial Network）跨阶部分连接网络，而 $CSPNet$ 引入的跨阶段部分连接机制是一种能在网络中不同层级之间建立直接连接的方法。这种连接方式有助于在不同层级之间传递信息，提高网络的信息流动性和特征表示能力。

![1703337223029.png](http://pic.moguw.top/i/2023/12/23/6586dd095c600.png)

​	在 $C2f$ 结构中的Bottleneck块中借鉴了 $ResNet$ 的思想，运用残差连接，使得中间卷积块中只需要学习细微的变化，解决了模型学习能力退化的问题。

​    同时， $C2f$ 结构也借鉴了 $YOLOv7$ 所使用的 $ELAN$ 结构

![1703337259187.png](http://pic.moguw.top/i/2023/12/23/6586dd2e114d4.png)

​	在 Backbone 的最后是一个 $SPPF$（快速空间金字塔池化）层，这个主要借鉴了何恺明在微软实验室提出来的$SPP$（空间金字塔池化）理论，$SPP$ 能够使得任意大小的多尺度特征图能够转换成固定大小的特征向量。$SPPF$ 用两三个小池化层取代了大池化层（例如k=13可用3个k=5的池化层取代），减少了运算量。

![1703337291244.png](http://pic.moguw.top/i/2023/12/23/6586dd4e0a457.png)

### Neck

​	在Neck中实现了多尺度特征融合； $YOLOv8$ 的颈部取消上行 $1\times1$ 卷积层CBS，并且 $YOLOv8$ 的颈部一样以 $C2f$ 取代了 $C3$，增强多尺度特征混合效果

![1703337329328.png](http://pic.moguw.top/i/2023/12/23/6586dd7375400.png)



### Head

Head 部分改动较大，换成了目前主流的解耦头结构，将分类和检测头分离，同时也从 Anchor-Based 换成了 Anchor-Free。使得模型可以不仅能够应用于目标检测还能够应用于图像分割和图片分类



### SiLU激活函数

$$SiLU=x*sigmoid(x)=\frac{x}{1+e^{-x}}$$

$SiLU$激活函数的优点：

1. 平滑性：$SiLU$ 是一种平滑函数，梯度容易求，有助于减少模型训练中的震荡和不稳定性。
2. 非线性：$SiLU$ 的非线性特性使得神经网络能够学习和表示更复杂的类型
3. 适应性：$SiLU$ 对于不同范围的输入值具有适应性反应。当输入值在负数范围时，$SiLU$ 的反应趋近于0，减少负数对模型的影响
4. 梯度保持：相比于$ReLU$，在输入值为正数时，$SiLU$ 梯度在接近0时，也能保持较高的值，这有助于更好的传播梯度，促进模型的训练和收敛
5. 表示能力：$SiLU$ 在保持输入值为正数的同时，对于大输入值有较强的反应提高模型的表示能力 

### 损失函数

  $YOLOv8$ 使用的损失函数是变焦损失函数（$Varifocal Loss$），它的优点是：

1. 分类损失考虑到 $IoU$
2. 合并计算分类信心+位置误差
3. 正样本与负样本差别处理

$$\mathrm{VFL}(p,q)=
\begin{aligned}
-\mathrm{ql}(\mathrm{qlog}(p)+(1-q)\log(1-p))&&q>0 \\
-\mathrm{ap}^\gamma\mathrm{log}(1-p)&&q=0
\end{aligned}$$

