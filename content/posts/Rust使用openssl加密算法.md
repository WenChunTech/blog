---
title: "Rust使用openssl加密算法"
subtitle: ""
date: 2022-10-31T23:06:19+08:00
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
  - rust
  - openssl
  - aes
categories:
  - Rust
  - Openssl
  - Crypto

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

# AES 加密算法

> AES(Advanced Encryption Standard)，全称：高级加密标准，是一种最常见的对称加密算法

## 配置 Rust Toml 文件

```toml
[dependencies]
openssl = { version = "0.10", features = ["vendored"] }
```

## 示例代码

```rust
use openssl::symm::{Cipher, Crypter, Mode};

fn main() {
    let key = "061cecfd897548208c76c04b6e7fb".as_bytes();
    let crypto_word: &mut Vec<u8> = &mut "keyword".as_bytes().to_vec();
    let block_size = Cipher::aes_128_cbc().block_size();
    // 添加填充
    pkcs7_padding(crypto_word, block_size);
    let mut output = vec![0; 1024];
    // 取16位密钥
    let mut encrypter = Crypter::new(Cipher::aes_128_ecb(), Mode::Encrypt, &key[..16], None).unwrap();
    match encrypter.update(&f, &mut output) {
        Ok(size) => {
            eprintln!("size is: {size}");
            println!("{:?}", &output[..size]);
            println!("{:02x?}", &output[..size]); // 转换为16进制
        }
        Err(_) => {}
    };
}

fn pkcs7_padding(data: &mut Vec<u8>, block_size: usize) {
    let padding_num = block_size - data.len() % block_size;
    let padding = padding_num as u8;
    data.append(&mut [padding].repeat(padding_num));

}

```

参考链接：

- [AES 加解密-CBC ECB - 独孤剑—宇枫 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xzj8023tp/p/12970790.html)

- [AES 加密(3)：AES 加密模式与填充 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/131324301)
