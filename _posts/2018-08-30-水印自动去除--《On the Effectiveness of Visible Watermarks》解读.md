---
layout:     post
title:      论文分享-水印自动去除
subtitle:  《On the Effectiveness of Visible Watermarks》解读
date:       2018-08-30
author:     zhanghaichao
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - 图像处理
    - 算法
    - 论文
---
>水印在日常生活中随处可见，它是一种保护图像图片版权的机制，防止未经许可或授权的使用；而自动去水印的计算机算法的存在却可使用户轻松获取无水印图像，这是由于当前的水印技术存在一个漏洞：水印通常被一致地添加到很多图像上，这种一致性可用于反转水印的处理过程。有鉴于此，谷歌在论文《On the Effectiveness of Visible Watermarks》中针对可泛化的多图像抠图算法，提出了可使水印足够鲁棒以免被从单个图像中去除的方法，而且还更具抵抗性，可以避免水印从图像集中大批量去除。谷歌在其博客中对论文成果做了更详实介绍。

### 1、简介
水印使用的标准做法是假设他们防止了消费者获取干净的图片，确保没有未经许可或授权的使用。然而，最近在 CVPR 2017 上出现的一篇名为《On The Effectiveness Of Visible Watermarks》的论文中，我们发现一种计算机算法可以越过这一保护，自动去除水印，使用户轻松获取不带水印的干净图像。
<br>  
看一下去除水印的效果:<br>
![去除水印demo](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/reconstructed.png?raw=true)
<br>
文中有理解错误之处，欢迎大家交流指出。
<br>
### 2、论文摘要

众所周知，水印在标记和保护版权上有着重要的作用。在文章中，四位作者并未用到深度学习方法对水印进行处理，而是单单采用了传统的图像处理和优化方法。通过对大量图像水印的一致性进行分析，从而自动检测水印和恢复原本图像。作者们提出了一种新型算法，通过输入图片集合，该算法能过分离“前景”（水印），“阿尔法层”（Alpha matte）和“背景”（原图），进而还原图像。以此同时，作者们还探究水印嵌入时，不同类型的不一致性，从而探讨了更高鲁棒性和安全性的加水印方案。使得可视水印不单单对单一图片的擦除具有高抵抗性，还使得对大规模图片集也能保持高抵抗性。
<br><br>

### 3、算法( An Attack on Watermarked Collections )

基于单张图像去除水印的难度还是很大的。所以论文中采用的方法分为三步：  
1. 搜集使用同一个水印的大量图像  
2. 基于这些图像，我们估计出 Watermark (W) 和 alpha matte (α)  
3. 再得到没有水印的原始图像。  
<br>
如下图所示
<br>
![步骤](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/teaser.jpg?raw=true)
<br>

**在了解完大致步骤之后我们需要对加了水印的图片进行数学建模。图片模型如下:**  
<br>
水印图像 J 通常通过将水印 W 叠加到自然图像 I 来获得。
<br><br>
![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc1.png?raw=true)
<br><br>
其中p =（x，y）是像素位置，并且α（p）是空间变化的不透明度或α无光泽。 最常用的水印是半透明的，以保持底层的图像内容部分可见。 也就是说，对于所有像素，α（p）<1，或者α= c·αn，其中c <1是常数混合因子，αn∈[0,1]是归一化的α遮罩。 类似于自然图像消光，对于αn，大多数像素只是背景（αn（p）= 0）或仅前景（αn（p）= 1）。
<br><br>
一旦得到了 W 和 α，那么原始图像就很容易得到:
<br><br>
![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc2.png?raw=true)
<br><br>
对于单张图像的情况，给定J 去得到 I 是很难的，因为 这个问题是 under-determined， 变量个数大于方程的个数（there are fewer equations than unknowns）
<br><br>
但是同一个水印通常以同一种方式被加到许多图像上去。对于一组使用了相同的 W 和α 的图像，可以用下面的公式表示 
<br><br>
![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc3.png?raw=true)
<br><br>
对于 K个彩色图像，每个像素有 3K 个方程 和 3(K + 1) + 1 个变量,这个多图像消光问题仍然没有被确定。  
但是因为 图像集中的 W 和α 的一致性，以及自然图像的先验知识，可以全自动的求解上述问题，得到很高精度的解。
<br><br>
我们去除水印的算法包括几个步骤，如下图所示： 
<br>
![图2.自动水印提取管道](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/Figure2.png?raw=true)
<br>
>(1)算法首先估计水印（alpha遮罩和水印图像），然后通过检测整个集合中所有图像的梯度将水印定位，除非空间变换，这个初始估计是正确的。  
(2)对齐的检测用于估计初始的alpha遮罩，并且细化估计水印层。  
(3)然后将它们用作我们的多图像消光优化的初始化。 
 
所以我们首先来解决使用相同的 W 和 α 加水印的问题，然后再考虑 使用不同的 W 和 α 加水印的问题

#### 3.1 初始水印估计和检测(Initial Watermark Estimation & Detection)
![ 图3.初始水印估计和检测。](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/Figure3.png?raw=true)

>（a）用户在单个图像中围绕水印提供了一个粗略的边界框（对于网络上的当前库存集，这不需要;参见文本）。  
（b）（a）的梯度大小。  
（c）水印检测和估计的2次迭代之后的集合中的中值梯度的幅度（见第3.1节）

I. 估计Matted水印(Estimating the Matted Watermark)  
第一步是对初始的水印位置的估计与检测，核心思想是计算水印图像梯度的中值，通过对 K 张图片的迭代，水印的梯度值会得到收敛，理论上当 K 越大收敛会越好，但事实是某一区间时会达到趋于平稳。对每个像素的x和y方向分别计算。
<br>
计算公式：
<br><br>
![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc4.png?raw=true)
<br><br>  
随着图像数K的增加， 4收敛到真正无水印的梯度Wm =αW，直到移位。 为了证明为什么会这样，我们将Ik和Jk视为随机变量，并计算异常E [∇Jk]。 使用方程 3我们有，

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc5.png?raw=true)

II. 水印检测(Watermark Detection)  
给定估计到的水印梯度 ，我们使用边缘模板匹配算法 Chamfer Distance 检测出所有图像中的水印位置.

>鉴于当前估计∇Wm，我们使用通常用于对象检测和识别中模板匹配的倒角距离来检测每个图像中的水印。 具体来说，对于给定的水印图像，我们获得一个详细的边缘图（使用Canny边缘检测器[4]），并计算其欧几里德距离变换，然后与∇Wm（水平和垂直翻转）卷积以获得倒角距离 从每个像素到最近的边缘。 最后，水印位置被认为是地图中距离最小的像素。 我们发现这种检测方法非常强大，为不同水印和不同的不透明度水平提供了很高的检测率 
为了初始化联合估计，如果水印在图像中的相对位置是固定的（如我们在网上观察到的任何股票图像集合的情况），我们得到水印梯度的初始估计∇Wm，由 相对于它们的中心注册图像并执行步骤I.如果水印位置不固定，则仅需要用户在其中一个图像中标记水印周围的粗略边界框。 然后，我们使用给定边界框中的梯度作为水印梯度的初始估计。 我们发现在步骤I和II之间进行迭代。 2-3次足以获得准确的检测和可靠的无水印水印估计。

#### 3.2 多图像消光与重建(Multi-Image Matting and Reconstruction) 
有了所有图像中水印的检测位置信息，我们的目标就是解决`多图像消光问题`，有 J 得到 W 、α、原始图像 I，我们定义如下的目标函数:

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc6.png?raw=true)

随后对公式中的每一项进行了解释

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc7.png?raw=true)

>ε= 0.001是逼近L1距离的鲁棒函数（为简洁起见，省略了p）。 
术语Ereg（∇I）和Ereg（∇W）是正则化项，它们鼓励重建图像和水印分段平滑，其中α亚光的梯度强。 将Ereg（∇I）定义为

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc8.png?raw=true)

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc9.png?raw=true)

**优化**

最后改写了目标函数的形式如下： 

>结果优化问题（方程6）是非线性的，当处理大集合（O（KN）不知道时，未知数的数量可能非常大，其中N和K是每个图像年龄和图像数量）。 为了应对这些挑战，我们引入辅助变量{Wk}，其中Wk是第k个图像的水印。 每个图像水印Wk需要接近W.原则上，我们重写如下的目标
 
![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc10.png?raw=true)

**使用下面的三个迭代步骤来求解上面的目标函数:**

I. 水印图片分解 (Image–Watermark Decomposition)  
这一步 通过固定 α 和 W 来最小化目标函数
  
>在这一步，我们最小化目标w.r.t. Wk和Ik，同时保持α 
和W固定。 因此，等式 10减少到：

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc11.png?raw=true)

>我们使用迭代 - 重叠 - 最小二乘法（IRLS）来解决这个最小化问题。

II. 水印更新 (Watermark Update)  
这一步 由所有图像的各自水印梯度得到全局水印梯度，也就是去中值 

>在此步骤中，我们选择通过最小化Eaux术语来估计与所有估计的每图像水印{Wk}一致的全局水印W。 这个步骤减少到取中位数{Wk}。 也就是说，W = mediank Wk。

III. 遮罩更新 (Matte Update)
这一步是通过固定其他参数，求解 α

>在这里，我们解决α，同时保持其他未知数固定。 在这种情况下，我们最小化以下目标超过α：

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/fc12.png?raw=true)

>这里也使用IRLS获得解决方案（最终的线性系统是在SM中得到的）。

**Matte and Blend Factor Initialization**

探讨了一些初始化的问题

#### 3.3 删除新图像中的水印( Removing the Watermark in a New Image ) 

有了 W and α 就可以将任意新图像中的水印去除（前提是使用了相同的 W and α）

>一旦我们得到了W和α的解决方案，我们可以使用它们从其标记的任何新图像中去除水印，而不需要再次运行整个流水线。非常微妙的水印或alpha哑光错误可能会显现出明显的视觉效果。 因此，我们避免直接重建和替代执行我们的多图像消隐算法的图像水印分解步骤（等式11）。

### 4、反水印去除—-广义水印模型(A Generalized Watermarking Model)
让水印不那么容易被去除，根据他去除水印的方法，破坏掉计算条件来达到反去除的目的

>一致的水印模型（等式3）是我们在网络上遇到的每个库存图像集合中实际使用的水印模型。 然而，一个自然的问题是是否可以通过破坏整个集合中水印的一致性来避免攻击。 为了获得视觉效果，我们探讨了从图像到图像的水印和alpha蒙板的变化的影响。

### 5、结果
- 股票图像的结果

>我们使用15种预定义查询（如“时尚”，“食物”，“运动”和“自然”）从流行的股票内容网站中搜索公开的预览图像。 
>在图像的一部分，水印区域是高度纹理的，而在其他图像中它是平滑的。 水印的不透明度低，并且在不同的数据集之间变化（估计的混合因子显示在每行之上）。 在所有这些情况下，我们的算法精确地估计了水印（图7）和原始图像（图6）。

- 对水印变化的鲁棒性

>我们评估了我们的广义框架对每图像水印变化的鲁棒性（见第4节）。 为此，我们生成了许多数据集，使用与之前相同的标志。 我们首先对每个图像的水印进行不同位置的均匀采样。 我们通过在全局混合因子c周围的几个强度水平下均匀采样每个图像的混合因子ck，即ck∈[c-x / 255，c + x / 255]（我们使用x = 10,20）。 我们通过用高斯滤波器平滑两个i.i.d随机噪声图像（用于扰动的x和y分量）来产生小的空间扰动。 我们将每个方向的最大扰动限制为一个定义的值（我们使用最多0.5和1个像素）。 然后我们使用那些作为位移字段来扭曲原始水印和alpha遮罩（使用双线性插值）。 最后，我们还使用上述变体的组合生成了数据集。

### 6、结论

- 发现了添加可见水印方式的漏洞，并且可以自动删除它们，然后以高精度恢复原始图像。
- 许多图像中的水印的一致性，并不受水印的复杂性或其在图像中的位置的限制。
- 去除水印最受几何变化的影响，通过这个特点，提出了改善水印安全性的方案

>图示出了攻击的示例限制。 特别是，当图像平滑时，估计微妙水印结构的不准确性偶尔会显示为可见的伪影。 我们推测，除了内容感知水印放置[12]之外，还可以利用这一事实来进一步提高去除的鲁棒性。

![](https://github.com/zhanghaichao520/zhanghaichao520.github.io/blob/master/post_img/2018-08-30-%E6%B0%B4%E5%8D%B0%E8%87%AA%E5%8A%A8%E5%8E%BB%E9%99%A4--%E3%80%8AOn%20the%20Effectiveness%20of%20Visible%20Watermarks%E3%80%8B%E8%A7%A3%E8%AF%BB/res.png?raw=true)

>水印变异的鲁棒性。 我们用几种类型的随机，每图像水印变化生成的水印数据集的检测率（超过所有图像），PNSR和DSSIM（平均超过50图像）：平移，不透明度，几何扰动及其组合。 请参阅文本中变化和幅度的解释。

### 研究这篇论文收集的资料：

1. [论文地址](http://openaccess.thecvf.com/content_cvpr_2017/papers/Dekel_On_the_Effectiveness_CVPR_2017_paper.pdf)

2. [项目页面](https://watermark-cvpr17.github.io/)

3. 论文翻译  
  - [论文的中文翻译1](http://blog.csdn.net/zhangjunhit/article/details/77680140?locationNum=2&fps=1)
  - [论文的中文翻译2](https://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2650729930&idx=3&sn=a4c759e819eeb9d33f7e749ad2ee0fca&chksm=871b29b4b06ca0a22b0492debe0f337fb1b0afd61b13b896ef492fadbf9201e28520e4bd5219&mpshare=1&scene=1&srcid=03176YwxO2ELrhO5sGiEU1Oh#rd)
  - [论文的中文翻译3](http://blog.csdn.net/zhuzhupozhuzhuxia/article/details/78027629)

4. 关于去除水印的GitHub源码

   - [link1](https://github.com/rohitrango/automatic-watermark-detection)

   - [link2](https://github.com/SixQuant/nowatermark)（去除效果不是很好.）

5. [视频智能去水印：从数学建模到工程实现。主要是从视频中提取图片，然后删除图片中的原有水印，在加上新的水印。](https://baijiahao.baidu.com/s?id=1585595777890022460&wfr=spider&for=pc)

