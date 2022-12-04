# Opencv抠图和裁剪图片


<!--more-->

# opencv实现抠图和裁剪



## 1. 抠图

**步骤**:

1. 加载图像
2. 转换图像格式(BGR --> HSV)
3. 设置阈值
4. 通过阈值提取部分区域
5. 显示图片

> 关于HSV可参考：[HSL和HSV色彩空间 - 维基百科，自由的百科全书 (wikipedia.org)](https://zh.wikipedia.org/wiki/HSL和HSV色彩空间)

```python
window_name = 'hsv'
img = cv2.imread(r"D:\picture\other\2E3246E873376135DC6F202D1456B37E.jpg")

# 设置高低阈值
hsv_low = np.array([0, 0, 0])
hsv_high = np.array([134, 255, 138])

# 将BGR转为HSV
dst = cv2.cvtColor(img, cv2.COLOR_HSV2BGR)
# 通过HSV的高低阈值，提取图像部分区域
mask_img = cv2.inRange(dst, hsv_low, hsv_high)

# 进行与位运算
img_ = cv2.bitwise_and(img, img, mask=mask_img)

cv2.imshow(window_name, img_)
cv2.waitKey(0)
cv2.destroyAllWindows()
```



## 2. 裁剪图片

**步骤**

1. 加载图像
2. 设置鼠标事件

		- 鼠标左键单击画点
		- 鼠标右键单击闭合所画的点
		- 鼠标右键双击填充		

```python
# 触发鼠标移动事件
# cv2.CV_EVENT_MOUSEMOVE
# 触发左键抬起事件
# cv2.CV_EVENT_LBUTTONUP
# 触发右键按下事件
# cv2.CV_EVENT_RBUTTONDOWN
# 触发右键抬起事件
# cv2.CV_EVENT_RBUTTONUP
# 触发左键双击事件
# cv2.CV_EVENT_LBUTTONDBLCLK
# 触发右键双击事件
# cv2.CV_EVENT_RBUTTONDBLCLK

coordinates = []
window_name = 'draw'

def on_mouse_callback(event, x, y, flag, param):
    # 左键点击,画点
    if event == cv2.EVENT_LBUTTONDOWN:
        xy = f'{x},{y}'
        coordinates.append((x, y))
        cv2.circle(img, (x, y), 1, (0, 0, 255), thickness=-1)
        cv2.putText(img,
                    xy, (x, y),
                    cv2.FONT_HERSHEY_PLAIN,
                    1.0, (0, 0, 0),
                    thickness=1)
        cv2.imshow(window_name, img)
    # 右键单击，画不规则图形
    elif event == cv2.EVENT_RBUTTONDOWN:
        pts = np.array(coordinates, np.int32)  # 顶点集
        #顶点坐标转为rowsx1x2, row为顶点数
        pts = pts.reshape((-1, 1, 2))
        cv2.polylines(img, [pts], True, (255, 255, 255), 2)
        cv2.imshow(window_name, img)

    # 右键双击，填充颜色
    elif event == cv2.EVENT_RBUTTONDBLCLK:
        area = np.array(coordinates)
        # 可以绘制多个图形
        cv2.fillPoly(img, [area], (255, 255, 255))
        # 绘制凸多边形
        # cv2.fillConvexPoly(img, area, (255, 255, 255))
        cv2.imshow(window_name, img)
        coordinates.clear()


cv2.namedWindow(window_name)
cv2.setMouseCallback(window_name, on_mouse_callback)
cv2.imshow(window_name, img)
cv2.waitKey(0)
cv2.destroyAllWindows()
```



# Rust版opencv实现抠图

Cargo.toml

```toml
[dependencies]
opencv = { version = "0.71" }
```



src/main.rs

```rust
use opencv::{
    core::{bitwise_and, in_range, Vector, CV_8UC3},
    highgui::{destroy_all_windows, imshow, wait_key},
    imgcodecs::{imread, ImreadModes},
    imgproc::{cvt_color, COLOR_BGR2HSV},
    prelude::*,
};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let img = imread(
        r"D:\picture\other\2E3246E873376135DC6F202D1456B37E.jpg",
        ImreadModes::IMREAD_COLOR as i32,
    )?;

    unsafe {
        let mut lowerb = Vector::from_slice(&[0, 0, 0u8]);
        let mut upperb = Vector::from_slice(&[134, 255, 138u8]);
        let mut dst = Mat::new_nd(img.dims(), &img.size().unwrap().width, CV_8UC3)?;
        let mut mask_img = Mat::new_nd(img.dims(), &img.size().unwrap().width, CV_8UC3)?;
        let mut result = Mat::new_nd(img.dims(), &img.size().unwrap().width, CV_8UC3)?;
        cvt_color(&img, &mut dst, COLOR_BGR2HSV, 0)?;
        in_range(&img, &mut lowerb, &mut upperb, &mut mask_img)?;
        bitwise_and(&img, &img, &mut result, &mask_img)?;
        imshow("winname", &result)?;
    }

    wait_key(0)?;
    destroy_all_windows()?;
    Ok(())
}

```





---

> 作者:   
> URL: https://release.blog-12x.pages.dev/opencv%E6%8A%A0%E5%9B%BE%E5%92%8C%E8%A3%81%E5%89%AA%E5%9B%BE%E7%89%87/  

