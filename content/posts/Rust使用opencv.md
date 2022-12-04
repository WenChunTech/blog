---
title: "Rust使用opencv"
subtitle: ""
date: 2022-12-04T13:30:26+08:00
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
- opencv
categories:
- Rust

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
# Rust中使用opencv
> 因为在macos和linux上安装比较简单，这里只介绍windows上的安装

## 安装环境

### 1. 安装opencv

**下载地址**：[Releases - OpenCV](https://opencv.org/releases/)

2. 选择windows平台，下载后默认安装即可

[![](https://pic.imgdb.cn/item/638c336b16f2c2beb1475b72.jpg)](https://pic.imgdb.cn/item/638c336b16f2c2beb1475b72.jpg)

3. 设置环境变量

- OPENCV_INCLUDE_PATHS
- OPENCV_LINK_LIBS
- OPENCV_LINK_PATHS

> 注意：opencv_world460这个不是固定的，需要根据下载的具体版本设置，我的版本中位于D:\development\opencv\build\x64\vc15\bin目录

[![](https://pic.imgdb.cn/item/638c343e16f2c2beb1489055.jpg)](https://pic.imgdb.cn/item/638c343e16f2c2beb1489055.jpg)

### 2. 安装LLVM

**下载地址：**[Releases · llvm/llvm-project (github.com)](https://github.com/llvm/llvm-project/releases)

根据需要安装32位或者64位，有些版本可能没有这2个选项，可以选择以前的版本，没必要选择最新的版本。下载后默认安装即可，注意需要在添加LLVM到环境变量中

![image-20221204135146944](C:\Users\FeiWenChun\AppData\Roaming\Typora\typora-user-images\image-20221204135146944.png)

## 简单使用

### 1. 确保安装了Rust环境

### 2. 新建项目

```powershell
cargo new opencv-rust-test
```

### 3. 配置Cargo.toml文件

```toml
[dependencies]
opencv = { version = "0.71" }
```

### 4. 将opencv_world460.dll文件拷贝到项目根目录(重要！！！)

如果opencv_world460.dll路径添加到环境变量中，应该就不需要将文件拷贝(没有实际测试)

### 5. 编写代码测试环境是否正常

src/main.rs

```rust
use opencv::{
    highgui::{destroy_all_windows, imshow, wait_key},
    imgcodecs::{imread, ImreadModes},
};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let img_path = r"D:\picture\other\2E3246E873376135DC6F202D1456B37E.jpg";
    let img = imread(img_path, ImreadModes::IMREAD_COLOR as i32)?;

    imshow("winname", &img)?;
    wait_key(0)?;
    destroy_all_windows()?;
    Ok(())
}

```

### 6. 运行测试

```powershell
# 使用cargo命令运行看看是否正常显示图片
cargo run
```

### 7. 报错解决

主要参考官方文档：

- [twistedfall/opencv-rust: Rust bindings for OpenCV 3 & 4 (github.com)](https://github.com/twistedfall/opencv-rust)
