>水印在日常生活中随处可见，它是一种保护图像图片版权的机制，防止未经许可或授权的使用；而自动去水印的计算机算法的存在却可使用户轻松获取无水印图像，这是由于当前的水印技术存在一个漏洞：水印通常被一致地添加到很多图像上，这种一致性可用于反转水印的处理过程。有鉴于此，谷歌在论文《On the Effectiveness of Visible Watermarks》中针对可泛化的多图像抠图算法，提出了可使水印足够鲁棒以免被从单个图像中去除的方法，而且还更具抵抗性，可以避免水印从图像集中大批量去除。谷歌在其博客中对论文成果做了更详实介绍。

###简介：
水印使用的标准做法是假设他们防止了消费者获取干净的图片，确保没有未经许可或授权的使用。然而，最近在 CVPR 2017 上出现的一篇名为《On The Effectiveness Of Visible Watermarks》的论文中，我们发现一种计算机算法可以越过这一保护，自动去除水印，使用户轻松获取不带水印的干净图像。  
![去除水印demo](https://watermark-cvpr17.github.io/supplemental/AdobeStock/image/AdobeStock_70306536_WM.jpg)
![去除水印demo](https://watermark-cvpr17.github.io/supplemental/AdobeStock/detection/AdobeStock_70306536_WM_detection.jpg)
![去除水印demo](https://watermark-cvpr17.github.io/supplemental/AdobeStock/reconstruction/AdobeStock_70306536_WM_recon.jpg)
![去除水印demo](https://watermark-cvpr17.github.io/images/teaser.jpg)
<center>cc</center>