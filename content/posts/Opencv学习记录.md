---
title: "Opencv学习记录"
subtitle: ""
date: 2023-01-03T22:43:23+08:00
draft: true
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
- draft
categories:
- draft

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---

<!--more-->

# Opencv使用记录

## 基本操作

### 1.访问和修改像素

```python
import cv2
img_path = ''
img = cv2.imread(img_path)

# shape属性
h,w,c = img.shape

# 图像总像素数
print(img.size)

# 图像数据类型
print(img.dtype)

#仅访问蓝色通道的像素
# 访问蓝色通道
blue = img[100,100,0]

#修改三通道的值
img[100,100] = [255,255,255]

# 更好的访问和编辑像素的方法
#访问 红色通道 的值
img.item(10,10,2)

#修改 红色通道 的值
img.itemset((10,10,2),100)
img.item(10,10,2)

# 合并和拆分通道
# 拆分通道
b,g,r = CV.spilt(img)

# 合并通道
img = CV.merge((b,g,r))
b = img[:,:,0]

#图像混合
dst = cv.addWeighted(img1,0.7,img2,0.3,0)
```



## 颜色空间

> 在 OpenCV 中有超过 150 种颜色空间转换的方法。但我们仅需要研究两个最常使用的方法，他们是 BGR 到 Gray，BGR 到 HSV。

使用 cv.cvtColor(input_image, flag)函数进行颜色转换，其中 flag 决定了转换的类型。

BGR 到 Gray 转换flag 为 **[cv.COLOR_BGR2GRAY](https://docs.opencv.org/4.0.0/d8/d01/group__imgproc__color__conversions.html#gga4e0972be5de079fed4e3a10e24ef5ef0a353a4b8db9040165db4dacb5bcefb6ea)**, BGR 到 HSV,  flag 为 **[cv.COLOR_BGR2HSV](https://docs.opencv.org/4.0.0/d8/d01/group__imgproc__color__conversions.html#gga4e0972be5de079fed4e3a10e24ef5ef0aa4a7f0ecf2e94150699e48c79139ee12)**。

**HSL**和**HSV**都是一种将[RGB色彩模型](https://zh.wikipedia.org/wiki/三原色光模式)中的点在[圆柱坐标系](https://zh.wikipedia.org/wiki/圓柱坐標系)中的表示法。这两种表示法试图做到比基于[笛卡尔坐标系](https://zh.wikipedia.org/wiki/笛卡尔坐标系)的几何结构RGB更加直观。

1. [HSV](https://zh.wikipedia.org/wiki/HSV色彩空间)（[色相](https://zh.wikipedia.org/wiki/色相)：**H**ue、[饱和度](https://zh.wikipedia.org/wiki/饱和度)：**S**aturation、[明度](https://zh.wikipedia.org/wiki/明度)；**V**alue），也称HSB（B指**B**rightness）是艺术家们常用的，因为与加法减法混色的术语相比，使用色相、饱和度等概念描述色彩更自然直观。HSV是RGB色彩空间的一种变形，它的内容与色彩尺度与其出处——RGB色彩空间有密切联系。
2. [HSL](https://zh.wikipedia.org/wiki/HSL色彩空间)（[色相](https://zh.wikipedia.org/wiki/色相)：**H**ue、[饱和度](https://zh.wikipedia.org/wiki/饱和度)：**S**aturation、[亮度](https://zh.wikipedia.org/wiki/亮度)：**L**ightness／**L**uminance），也称HLS或HSI（I指**I**ntensity）与HSV非常相似，仅用亮度（Lightness）替代了明度（Brightness）。二者区别在于，一种纯色的明度等于白色的明度，而纯色的亮度等于中度灰的亮度。

- [色相](https://zh.wikipedia.org/wiki/色相)（H）是色彩的基本属性，就是平常所说的[颜色](https://zh.wikipedia.org/wiki/颜色)名称，如[红色](https://zh.wikipedia.org/wiki/红色)、[黄色](https://zh.wikipedia.org/wiki/黄色)等。

- [饱和度](https://zh.wikipedia.org/wiki/色度_(色彩学))（S）是指色彩的纯度，越高色彩越纯，低则逐渐变灰，取0-100%的数值。
- [明度](https://zh.wikipedia.org/wiki/明度)（V），亮度（L），取0-100%。

### 从RGB到HSL或HSV的转换

设 (*r*, *g*, *b*)分别是一个颜色的红、绿和蓝坐标，它们的值是在0到1之间的实数。设*max*等价于*r*, *g*和*b*中的[最大者](https://zh.wikipedia.org/wiki/极值)。设*min*等于这些值中的最小者。要找到在HSL空间中的 (*h*, *s*, *l*)值，这里的*h* ∈ [0, 360）[度](https://zh.wikipedia.org/wiki/角度)是角度的色相角，而*s*, *l* ∈ [0,1]是饱和度和亮度，计算为：
$$
h = \begin{cases}
0^o, if max=min\\
60^o \times \frac{g-b}{max-min} + 0^o,if \space max=r and g \ge b\\
60^o \times \frac{g-b}{max-min} + 360^o,if \space max=r and g \lt b\\
60^o \times \frac{b-r}{max-min} + 120^o,if \space max=g\\
60^o \times \frac{r-g}{max-min} + 240^o,if \space max=b\\
\end{cases}
$$

$$
s = \begin{cases}
0,if \space l = 0 \space or \space max = min\\
\frac{max-min}{max+min}= \frac{max-min}{2l}, if \space 0 < l \le \frac{1}{2}\\
\frac{max-min}{2-(max+min)} = \frac{max-min}{2-2l}, if \space l > \frac{1}{2}
\end{cases} \\
l = \frac{1}{2} \times (max+min)
$$

*h*的值通常规范化到位于0到360°之间。而*h* = 0用于*max* = *min*的（定义为灰色）时候而不是留下*h*未定义。

HSL和HSV有同样的[色相](https://zh.wikipedia.org/wiki/色相)定义，但是其他分量不同。HSV颜色的*s*和*v*的值定义如下：
$$
s = \begin{cases}
0, if \space max = 0\\
\frac{max-min}{max} = 1 - \frac{min}{max}, otherwise
\end{cases}
$$




## 几何变换

### 缩放

1. 函数签名: `cv2.resize(src,dsize,dst=None,fx=None,fy=None,interpolation=None)`

	- **InputArray src** ：输入，原图像，即待改变大小的图像；

	- **OutputArray dst**： 输出，改变后的图像。这个图像和原图像具有相同的内容，只是大小和原图像不一样而已；
	- **dsize**：输出图像的大小，如上面例子（300，300）。
	- **fx**：width方向的缩放比例,如果是 0，按照 `dsize * width/src.cols` 计算
	- **fy**：height方向的缩放比例,如果是 0，按照 `dsize * height/src.rows` 计算
	- **interpolation(插值)**：这个是指定插值的方式，图像缩放之后，肯定像素要进行重新计算的，就靠这个参数来指定重新计算像素的方式，有以下几种：
		- INTER_NEAREST - 最邻近插值
		- INTER_LINEAR - 双线性插值，如果最后一个参数你不指定，默认使用这种方法
		- INTER_CUBIC - 4x4像素邻域内的双立方插值
		- INTER_LANCZOS4 - 8x8像素邻域内的Lanczos插值
1.  原理:

  - 最邻近插值: 

$$
\text{计,原图片坐标为(i,j),原图片大小为$n \times n $,转化后的图像大小为$m \times m$},x,y分别为转换后的坐标\\
x = \frac{m}{n} \times i \\
y = \frac{m}{n} \times j \\
$$



  - 双线性插值(默认)

[![](https://pic.imgdb.cn/item/63b6e4f7be43e0d30e99aa5d.png)](https://pic.imgdb.cn/item/63b6e4f7be43e0d30e99aa5d.png)

假如我们想得到未知函数 f 在点 P=(x, y) 的值，假设我们已知函数f 在Q11 = （x1, y1），*Q*12 = (*x*1,*y*2)*，Q*21 = (*x*2,*y*1) 以及*Q*22 = (*x*2,*y*2) 四个点的值。

　　首先在x方向进行线性插值，得到R1， R2，然后在y方向进行线性插值，得到P。这样就得到所要的结果 f(x, y)

　　其中红色点Q11, Q12, Q21, Q22为已知的4个像素点。

　　第一步：X方向的线性插值，在Q12， Q22中插入蓝色点R2， Q11， Q21 中插入蓝色点R1

　　第二步：Y方向的线性插值，通过第一步计算出的R1与R2在y方向上的插值计算出P点。

　　线性插值的结果与插值的顺序无关，首先进行y方向的插值，然后进行X方向的插值，所得到的结果是一样的，双线性插值的结果与先进行哪个方向的插值无关。

​		在X方向上：
$$
f(R_1) \approx \frac{x_2-x}{x_2-x_1} f(Q_{11}) + \frac{x-x_1}{x_2-x_1}f(Q_{21}) \space where \space R_1 = (x,y_1) \\
f(R_2) \approx \frac{x_2-x}{x_2-x_1} f(Q_{12}) + \frac{x-x_1}{x_2-x_1}f(Q_{22}) \space where \space R_1 = (x,y_2)
$$
​		在y方向上:
$$
f(P) \approx \frac{y_2-y}{y_2-y_1}f(R_1) + \frac{y-y_1}{y_2-y_1}f(R_2)
$$


### 平移

### 旋转

### 仿射变换

### 透视变换

## 形态学操作

## 边缘检测

## 图像金字塔

## 角点检测





**参考资料：**

- [OpenCV中文官方文档 (woshicver.com)](http://woshicver.com/)
- [https://opencv.apachecn.org](https://opencv.apachecn.org/#/)
- [HSL和HSV色彩空间 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/HSL和HSV色彩空间)
- [cv2.resize()原理详解_AI bro的博客-CSDN博客_cv2.resize](https://blog.csdn.net/weixin_41466575/article/details/113058802)
