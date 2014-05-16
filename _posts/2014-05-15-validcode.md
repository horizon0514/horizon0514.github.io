---
layout: post
title: "破解北邮研究生教务系统登陆验证码"
description: ""
category: 
tags: [python,图像处理]
---


**在**北邮人论坛看到一个帖子——[《看看人家的教务处》](http://bbs.byr.cn/#!article/Picture/2972823)，加上自己之前也开发过微信公众号，虽然很渣的水平，但还是想照猫画虎一番。

首先就要模拟登陆[北邮研究生教务系统](http://yjxt.bupt.edu.cn)，利用python的request的模块可以很容易的实现，可是问题来了。登陆界面使用了图片验证码。这可怎么办？从来没遇到过这种情况啊。于是一番Google，找到了一些[博文](http://xiaoxia.org/2011/05/31/boring-entry-the-fabled-verification-code-recognition-technology-learning-notes/).

总的来说，简单的验证码识别分为3步。
* 大量抓取验证码，人工识别，建立字模库
* 将待识别的验证码去除噪声，分片。
* 与字模比对，统计处像素点不同的个数。不同数最低的相似度最高。得出结果


**第一步**提取字模。

使用Chrome的审查元素得出验证码的链接为http://yjxt.bupt.edu.cn/Public/ValidateCode.aspx ，利用request 及 PIL 模块抓取50张验证码图片。代码如下：

```python


#!/usr/bin/env python
# coding=utf-8
import requests
from PIL import Image
from StringIO import StringIO
for i in range(50):
    url = 'http://yjxt.bupt.edu.cn/Public/ValidateCode.aspx'
    r = requests.get(url)
    img = Image.open(StringIO(r.content))
    img.save("%s.png"%i)

```
发现这些验证码都是很简单、清楚、规范的四位阿拉伯数字，顿时松了一口气。对这些图片进行分片提取字模。利用ubuntu自带的GIMP Image Editor 可以看出，图片是52 × 10像素的，然后一个一个数像素点，第一个数字最左侧非空白像素x坐标为5，最右侧x坐标为15，最上方y坐标为4，底部y坐标为19。每个数字间距为19。有了这些坐标值，我们就可以构建一个个的“盒子”把四个数字包围起来，也就是分片。代码如下：

```python
#!/usr/bin/env python
# coding=utf-8

from PIL import ImageEnhance,ImageFilter

def splitImage(img):
    leftFirst=5
    rightFirst=15
    width=12
    top=4
    bottom=19

    img = img.filter(ImageFilter.MedianFilter())
    img_denoise = ImageEnhance.Contrast(img)
    img_denoise = img_denoise.enhance(3.0)   #去噪

    imgs = {}
    left = []
    right = []
    
    for i in range(4):
        left.append(leftFirst+i*width)
        right.append(rightFirst+i*width)
        box = (left[i],top,right[i],bottom)
        imgs[i] = img_denoise.crop(box)

    return imgs

```








