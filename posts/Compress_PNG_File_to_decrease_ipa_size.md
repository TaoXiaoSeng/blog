---
title: 如果处理PNG图片，从而降低IPA包大小以及提高图片使用时的性能
date: '2014-02-14'
description: 如果通过压缩PNG图片，降低ipa包的大小。
permalink: '/2014/02/14/compress_png_file_to_decrease_ipa_size.html'
categories:
- xcode
tags:
- xcode

---

## 降低包大小
1：使用图片压缩工具ImageOptm来对图片进行压缩，该工具会使用多种算法挑选压缩后大小最小的一张图输出。
2：可以考虑使用jpg图片
3：如果手工自己压缩图片，则最好不要关闭xcode工程里自带的png图片压缩选项，即targets-> Build Settings->Compress PNG Files设置为NO，否则可能引起一些已经被手动压缩过的图片经过xcode处理后，再次变大了一些。注意这个选项只适用于png图片
4：如果一张图可以通过拉伸等处理得到最后的效果，可以考虑切图的时候以九宫格的方式切小图。


## 提高图片使用性能：
1：在图片质量可以接受的情况下，图片越小越好
2：如果不要使用透明度，则可以考虑图片不带有alpha通道。
3：图片的大小最好等于UI显示时的尺寸，否则图片缩放处理会消耗一定的性能（这里涉及了矩阵运算）
4：图片draw的时候，最好可以设置draw的时候图片不透明。
5：显示图片的时候最好像素对齐，能降低抗锯齿的计算。