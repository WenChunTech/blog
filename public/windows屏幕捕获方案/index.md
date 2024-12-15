# Windows屏幕捕获方案


<!--more-->

# Windows 屏幕捕获方案

## Windows GDI
  这是 Windows 系统上兼容性最好的一套方案，从 Windows 2000 开始提供，部分接口底层支持了硬件加速功能。

## Microsoft DirectX Graphics Infrastructure（DXGI）
  这套 API 是为了封装不同版本 DirectX 部分接口而出现的，首次出现是在 Direct 3D 10，性能也不错。

## DWM
  这项技术是微软在 Vista 上，为了提高桌面窗口显示效果而提供的一套 API，其基本思路是：改变过去直接绘制窗口到屏幕的做法，在视频内存中开辟一块离屏渲染区域，用来做各种像素处理，之后再交给屏幕显示，以此实现 Vista 有的毛玻璃效果，以及按下窗口键 +Tab 键来做桌面窗口切换时的 3D 转化效果等。**通过这套 API 我们可以实现窗口缩略图的渲染，**在 Windows 7 上，DWM 可以通过切换系统主题来控制其打开和关闭，即 Aero 主题。从 Windows 8 开始，系统默认都是打开 DWM 。这套 API 一般可以用来获取目标窗口的缩略图（thumbnail，支持动态更新），用来分享画面的话，分辨率偏低，不太能满足需求。

## Magnification API
  我们姑且称之为“放大镜 API”，这个是 Windows 上预装的放大镜程序底层使用的技术，我们正好可以利用这个技术，同时还可以实现窗口过滤功能。这套 API 也是目前兼容范围较广，各类 App 使用最多的窗口显示数据抓取技术，我们后面会详细分析。

## Windows Grapics Capture（WGC）
  微软最新提供的一套正式用来抓取屏幕数据的接口，遗憾的是最初是给 UWP 提供的，而非 Win32 App，且要求必须是 Windows 10, Update (1809) 及之后的系统才支持，但是官方提供的，我们也要予以足够的重视。

## Windows Media API 以及各种 hook 技术
  可用范围较窄，因此这里不讨论。



## 各个方案对比：

| 方案              | 性能 | 兼容性                                                                      | 是否支持指定窗口 |
| ----------------- | ---- | --------------------------------------------------------------------------- | ---------------- |
| GDI               | 一般 | Windows 2000 之后提供，兼容性最好                                           | 支持             |
| DXGI              | 高   |                                                                             | 不支持           |
| DWN               | 低   |                                                                             |                  |
| Magnification API |      |                                                                             | 支持             |
| WGC               | 高   | 支持 Windows 10， Update (1809) （时间是2018-11-13） 以及更高版本的操作系统 | 支持             |



### 参考链接

- [【技术分享】Windows桌面端录屏采集实现教程](https://doc-markdown-zh.zego.im/ji-zhu-fen-xiang-windowszhuo-mian-duan-lu-ping-cai-ji-shi-xian-jiao-cheng/)
- [技术干货 | 桌面端屏幕分享实践 - 脉脉](https://maimai.cn/article/detail?fid=1692948703&efid=cOHX1WHd8IoUNDbp54IhUw)
- [Windows桌面共享中一些常见的抓屏技术 - 厚积薄发 - C++博客](http://www.cppblog.com/weiym/archive/2016/08/11/204536.html)

---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/windows%E5%B1%8F%E5%B9%95%E6%8D%95%E8%8E%B7%E6%96%B9%E6%A1%88/  

