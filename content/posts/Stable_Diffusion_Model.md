---
title: Stable_Diffusion_Model
subtitle:
date: 2023-03-30T23:16:41+08:00
draft: true
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - draft
categories:
  - draft
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---

<!--more-->
# Stable Diffusion Model

## 一. 原理

1. diffusion整体思路如下:

![](https://pic.imgdb.cn/item/6429a893a682492fcc103461.jpg)

说明: 整个过程主要分为正向过程和逆向过程.正向过程主要是将图像转化为纯噪声的过程,而逆向过程正好相反,是将纯噪声还原为原图像的过程.

- 正向过程:对于一张图像$\alpha_0$我们为它添加一个服从标准正态分布的噪声$z_0$,然后再在此基础上添加噪声$z_1$,每次添加的噪声都比上一次添加的噪声多,重复此操作,直到变为纯噪声$z_n$,此过程可以引出公式:

    $\alpha_t = 1 - \beta_t$

    $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t} z_1$,

    其中$\beta$需要越来越大,从而$\alpha$越来越小,可以将$\sqrt{1-\alpha_t}$理解为噪声的权重,这样每次生成的噪声都比上一次多

- 逆向过程:需要生成一个服从标准正态分布的噪声,然后再在此基础上进行去噪,得到上一步的图像,重复此操作得到最原始的图像$x_0$

2. 实现细节如下:

    - 正向过程:可以思考一下在$x_1$时刻的图像受到那些因素影响呢?答案很明显,受到前一刻图像和所加噪声的影响.

        从公式:

        $\alpha_t = 1 - \beta_t$

        $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_t} z_t$

        可以明显看出来.按照这个思路我们可以一步一步从$x_0 \rightarrow x_1 \rightarrow x_2 \rightarrow ... \rightarrow x_n$生成.那这样是不是计算量很大呢?有没有简单的方法直接从$x_0$生成到$x_n$呢?答案是可以的.下面详细推导一下.

        已知:

        ​    $x_t = \sqrt{\alpha_t}x_{t-1} + \sqrt{1-\alpha_{t-1}} z_t$

        ​    $若x \sim \mathcal N(\mu, \sigma^2), 则\alpha x \sim \mathcal N(\alpha\mu, (\alpha\sigma)^2)$

        $若x \sim \mathcal N(\mu_1,\sigma_1^2),y \sim \mathcal N(\mu_2, \sigma^2_2),则x+y \sim \mathcal N(\mu_1+\mu_2,\sigma_1^2+\sigma_2^2)$

        ​    $x_{t-1} = \sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}} z_{t-1}$

        可推导出:

        $$\begin{split}
        x_t &= \sqrt{\alpha_t}(\sqrt{\alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_{t-1}}z_{t-2}) + \sqrt{1-\alpha_t}z_{t-1} \\
        &= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{\alpha_t (1-\alpha_{t-1})}z_{t-2}) + \sqrt{1-\alpha_t}z_{t-1} \\
        &= \sqrt{\alpha_t \alpha_{t-1}}x_{t-2} + \sqrt{1-\alpha_t \alpha_{t-1}} \hat {z})\\ \vdots \\ &= \sqrt{\bar \alpha_t}x_0 + \sqrt{1-\bar \alpha_t} \hat z \end{split}$$

        其中: $\bar \alpha_t$表示累乘即$\prod \limits_{i=0} \limits^t {\alpha_i}$, $\hat z$表示服从**标准正态分布**

        现在详细讲解关键推导过程:

        已知:

        ​    $\mathcal N( \mu, \sigma^2) = \mathcal N(0, 1) 标准正态分布$

        ​    $z_0,z_1, ..., z_n \sim \mathcal N(0, 1)$

        则:

        ​    $\sqrt{\alpha_t(1-\alpha_{t-1})}z_{t-2}) \sim \mathcal N(0, \alpha_t(1-\alpha_{t-1}) $

        ​    $\sqrt{1-\alpha_t}z_{t-1} \sim \mathcal N(0, 1-\alpha_t)$

        根据正态分布性质,上面2式相加可得:

        ​    $\sqrt{1-\alpha_t \alpha_{t-1}} \hat {z}) \sim \mathcal N(0, 1-\alpha_t \alpha_{t-1})$

        由此可知$\hat z $也是服从标准正态分布.

    - 逆向过程:需要从服从标准正态分布的噪声$x_t$还原成原始图片,现在我们已知条件有什么呢?只有$x_t$.那我们能像正向过程哪样从$x_t$直接推导出$x_0$我们需要的图像吗?答案是不行的,那我们只有一步一步的向前推导,直到推导出$x_t$.那现在我们需要怎么从$x_t$推导出$x_{t-1}$.

        需要用到的方法是贝叶斯公式:

        $P(A|B) = \frac {P(B|A) \cdot {P(A)}} {P(B)}$

        那么我们用贝叶斯公式表示$x_{t-1}$就是:

        $q(x_{t-1}| x_t) = q(x_{t}|x_{t-1}) \frac {q(x_{t-1})} {q(x_t)}$

        其中$q(x_{t}|x_{t-1})$我们可以从正向过程推导出的公式可以求出.但是$q(x_{t-1})$和$q(x_t)$我们是不知道的.但是由正向过程推导的公式可知可以由$x_0$推导任意时刻的$x_t$,所以可以为上式添加一个条件作为一个已知条件,即:

        $q(x_{t-1}| x_t, x_0) = q(x_{t}|x_{t-1}) \frac {q(x_{t-1}|x_0)} {q(x_t|x_0)}$

        现在我们可以由上式可以推导出:

        $q(x_{t-1}|x_0) = \sqrt {\bar \alpha_{t-1}}x_0 + \sqrt {1- \bar \alpha_{t-1}}z \sim \mathcal N(\sqrt {\bar \alpha_{t-1}}x_0, 1- \bar\alpha_{t-1})$

        $q(x_{t}|x_0) = \sqrt {\bar \alpha_{t}}x_0 + \sqrt {1- \bar \alpha_{t}}z \sim \mathcal N(\sqrt {\bar \alpha_{t}}x_0, 1- \bar\alpha_{t})$

        $q(x_{t}|x_{t-1},x_0) = \sqrt {\alpha_{t}}x_{t-1} + \sqrt {1-\alpha_{t}}z \sim \mathcal N(\sqrt {\alpha_{t}}x_{t-1}, 1-\alpha_{t})$

        我们已知正态分布的表达式为:

        $f(x) = \frac {1} {\sqrt {2 \pi \sigma}} e^{-\frac{(x-\mu)^2} {2\sigma^2}}$

        所以:

        $q(x_{t-1}|x_0) = \frac{1} {\sqrt{2\pi \sqrt{1-\bar \alpha_{t-1}}}} e^{-\frac{(x-\sqrt{\bar \alpha_{t-1}}x_0)^2} {2(1-\bar\alpha_{t-1})}}$

        $q(x_{t}|x_0) = \frac{1} {\sqrt{2\pi \sqrt{1-\bar \alpha_{t}}}} e^{-\frac{(x-\sqrt{\bar \alpha_{t}}x_0)^2} {2(1-\bar\alpha_{t})}}$

        $q(x_{t}|x_{t-1},x_0) = \frac{1} {\sqrt{2\pi \sqrt{1-\alpha_{t}}}} e^{-\frac{(x-\sqrt{\alpha_{t}}x_{t-1})^2} {2(1-\alpha_{t})}}$

        由上面3个公式可得:

        $q(x_{t-1}| x_t, x_0) = \frac { \frac{1} {\sqrt{2\pi \sqrt{1-\bar \alpha_{t-1}}}} \cdot  \frac{1} {\sqrt{2\pi \sqrt{1-\bar \alpha_{t}}}}} {\frac{1} {\sqrt{2\pi \sqrt{1-\alpha_{t}}}} } e^{-\frac{1}{2} (-\frac{(x-\sqrt{\bar \alpha_{t-1}}x_0)^2} {1-\bar\alpha_{t-1}} + \frac{(x-\sqrt{\bar \alpha_{t}}x_0)^2} {1-\bar\alpha_{t}} - \frac{(x-\sqrt{\alpha_{t}}x_{t-1})^2} {1-\alpha_{t}}) }$

        我们记$e$左边的值为$M$,则:
        $$\begin{split}q(x_{t-1}| x_t, x_0) &= M e^{-\frac{1}{2} (-\frac{(x-\sqrt{\bar \alpha_{t-1}}x_0)^2} {1-\bar\alpha_{t-1}} + \frac{(x-\sqrt{\bar \alpha_{t}}x_0)^2} {1-\bar\alpha_{t}} - \frac{(x-\sqrt{\alpha_{t}}x_{t-1})^2} {1-\alpha_{t}}) }\\
        &=Me^{-\frac{1}{2}[(\frac{\alpha_t}{\beta{t}}+ \frac{1}{1-\bar\alpha_{t-1}})x_{t-1}^2-(\frac{2\sqrt{\bar\alpha_t}}{1-\bar\alpha}x_0)x_{t-1}+c(x_t,x_0)]} \end{split}$$

        因为我们不关心$x_t,x_0$的值,我们直接用$c(x_t,x_0)$表示,我们推导出这个公式主要就是**求出均值($\mu$)和方差($\sigma$)**,我们知道**正态分布进行乘除运算仍然符合正态分布**,也就是说$q(x_{t-1}|x_t)$符合正态分布,则:
        $$\begin{split}
        f(x) &= \frac {1} {\sqrt {2 \pi \sigma}} e^{-\frac{(x-\mu)^2} {2\sigma^2}} \\
        &= \frac {1} {\sqrt {2 \pi \sigma}} e^{-\frac{1}{2}(\frac{x^2}{\sigma^2} - \frac{2\mu x} {\sigma^2} + \frac{\mu^2}{\sigma^2})}
        \end{split}$$

        由此可以建立方程组:

        $$\begin{cases}
        \frac{\alpha_t}{\beta{t}}+ \frac{1}{1-\bar\alpha_{t-1}} = \frac{1}{\sigma^2} \\
        \frac{2\sqrt{\bar\alpha_t}}{1-\bar\alpha}x_0 = \frac{2\mu}{\sigma^2}
        \end{cases}$$

        解得:

        $$\begin{cases}
        \sigma^2 &= \frac{\beta_t(1-\bar\alpha_{t-1})} {1-\bar\alpha_t} \\
        \mu &= \frac{\sqrt{\alpha_t(1-\bar\alpha_{t-1})}} {1-\bar\alpha_t}x_t + \frac{\sqrt{\bar\alpha_{t-1}}\beta_t} {1-\bar\alpha_t}x_0
        \end{cases}$$

        现在我们求出了均值($\mu$)和方差($\sigma$),就可以根据这2个值求出$x_{t-1}$时刻的图像,但是现在有一个问题,那就是$x_0$我们是不知道的,它就是我们最后需要求出的值,这里我们可以从正向过程推导出的公式中反推$x_0$,即:

        $x_0 = \sqrt{\bar\alpha_t}(x_t - \sqrt{1-\bar\alpha} \hat z_t)$

        这样可以得到**$x_0$的估计值,计算后得到估计的$\hat\mu$,**最后可以得到:

        $$\begin{cases}
        \sigma^2 &= \frac{\beta_t(1-\bar\alpha_{t-1})} {1-\bar\alpha_t} \\
        \hat\mu &= \frac{1} {\sqrt{\alpha_t}} (x_t - \frac{\beta_t} {\sqrt{1-\bar\alpha_t}}\hat z_t)
        \end{cases}$$



## 二. 代码示例

**代码流程:**

![](https://img-blog.csdnimg.cn/img_convert/2d6c01c0cba8e6bb18891a7a2288f95f.png)







```python
class Diffusion:
    def __init__(self, noise_steps=1000, beta_start=1e-4, beta_end=0.02, img_size=256, device="cuda"):
        # define the noise steps, and that is the number of the standard normal distribution
        self.noise_steps = noise_steps
        # define the beta start value
        self.beta_start = beta_start
        # define the beta end value
        self.beta_end = beta_end
        # define generate the image size
        self.img_size = img_size
        # define the device to running
        self.device = device
        # define the noise schedule for per step
        self.beta = self.prepare_noise_schedule().to(device)
        # define the alpha value
        # $\alpha = 1 - \beta$
        self.alpha = 1. - self.beta
        # \hat \alpha = \alpha_0 \ctime \alpha_1 \ctime \alpha_2 ... \alpha_t
        self.alpha_hat = torch.cumprod(self.alpha, dim=0)

    def prepare_noise_schedule(self):
        return torch.linspace(self.beta_start, self.beta_end, self.noise_steps)

    def noise_images(self, x, t):
        sqrt_alpha_hat = torch.sqrt(self.alpha_hat[t])[:, None, None, None]
        sqrt_one_minus_alpha_hat = torch.sqrt(1 - self.alpha_hat[t])[:, None, None, None]
        Ɛ = torch.randn_like(x)
        return sqrt_alpha_hat * x + sqrt_one_minus_alpha_hat * Ɛ, Ɛ

    # define random the sample for per step
    def sample_timesteps(self, n):
        return torch.randint(low=1, high=self.noise_steps, size=(n,))

    def sample(self, model, n):
        logging.info(f"Sampling {n} new images....")
        model.eval()
        with torch.no_grad():
            # generate the random noise image
            x = torch.randn((n, 3, self.img_size, self.img_size)).to(self.device)
            for i in tqdm(reversed(range(1, self.noise_steps)), position=0):
                t = (torch.ones(n) * i).long().to(self.device)
                predicted_noise = model(x, t)
                alpha = self.alpha[t][:, None, None, None]
                alpha_hat = self.alpha_hat[t][:, None, None, None]
                beta = self.beta[t][:, None, None, None]
                if i > 1:
                    noise = torch.randn_like(x)
                else:
                    noise = torch.zeros_like(x)
                x = 1 / torch.sqrt(alpha) * (x - ((1 - alpha) / (torch.sqrt(1 - alpha_hat))) * predicted_noise) + torch.sqrt(beta) * noise
        model.train()
        x = (x.clamp(-1, 1) + 1) / 2
        x = (x * 255).type(torch.uint8)
        return x
```

**参考链接:**

- [Diffusion Model原理详解及源码解析_秃头小苏的博客-CSDN博客](https://blog.csdn.net/qq_47233366/article/details/128508412)
- [dome272/Diffusion-Models-pytorch: Pytorch implementation of Diffusion Models (https://arxiv.org/pdf/2006.11239.pdf) (github.com)](https://github.com/dome272/Diffusion-Models-pytorch)
