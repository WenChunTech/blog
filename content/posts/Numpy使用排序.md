---
title: "Numpy使用排序"
subtitle: ""
date: 2022-11-05T21:37:49+08:00
draft: false
author: ""
authorLink: ""
authorEmail: ""
description: ""
keywords: ""
license: ""
comment: false
weight: 0

tags:
  - numpy
  - sort
categories:
  - Numpy
  - Python

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

# Numpy 排序函数

1. `numpy.sort(a, axis=-1, kind=None, order=None)`

参数:

- a : 要排序的数组；
- axis ： 按什么轴进行排序，默认按最后一个轴进行排序；
- kind ：排序方法，默认是快速排序(不稳定)，可选参数有:{'quicksort', 'mergesort', 'heapsort', 'stable'}
- order : 当数组定义了字段属性时，可以按照某个属性进行排序；

[![](https://pic1.imgdb.cn/item/63666c7916f2c2beb17e0c00.jpg)](https://pic1.imgdb.cn/item/63666c7916f2c2beb17e0c00.jpg)

2. `numpy.argsort(a, axis=-1, kind=None, order=None)`:numpy.argsort 函数用于将数组排序后，返回数组元素从小到大依次排序的所有元素索引

参数:

- a : 要排序的数组
- axis ： 按什么轴进行排序，默认按最后一个轴进行排序
- kind ：排序方法，默认是快速排序
- order : 当数组定义了字段属性时，可以按照某个属性进行排序

[![](https://pic1.imgdb.cn/item/63666da616f2c2beb182c6fc.jpg)](https://pic1.imgdb.cn/item/63666da616f2c2beb182c6fc.jpg)

**排序后索引解释**： 比如说第一行排序后的结果为：`[1, 0, 2]` 表示原数组索引为 1 的数现在变为 0，原索引为 0 变为 1，原索引为 2 保持不变,即`[0, 1, 2] -> [1, 0, 2]`

3. `numpy.lexsort(keys, axis=-1)`: numpy.lexsort 函数用于按照多个条件（键）进行排序，返回排序后索引。
   > 这里举一个应用场景：小升初考试，重点班录取学生按照总成绩录取。在总成绩相同时，数学成绩高的优先录取，在总成绩和数学成绩都相同时，按照英语成绩录取…… 这里，总成绩排在电子表格的最后一列，数学成绩在倒数第二列，英语成绩在倒数第三列。

参数:

- keys ：序列或元组，要排序的不同的列
- axis ：指定次排序的轴(默认为-1，即最后一个轴)
  > 注意：根据数组的 shape 维数指定次排序轴，也就是说如果是二维数组只能设置为 0，因为 1 是著排序轴。如果 shape 为(2,3,5),则 axis 可指定为 0 或者 1

[![](https://pic1.imgdb.cn/item/636672ca16f2c2beb1911d97.jpg)](https://pic1.imgdb.cn/item/636672ca16f2c2beb1911d97.jpg)

**根据行和列排序**

[![](https://pic1.imgdb.cn/item/63667d7c16f2c2beb1c1680f.jpg)](https://pic1.imgdb.cn/item/63667d7c16f2c2beb1c1680f.jpg)
