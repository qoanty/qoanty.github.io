---
title: "极验滑动验证码识别"
date: 2019-05-07T09:24:08+08:00
tags: ["python", "geetest"]
categories: ["Captcha", "Python"]
draft: false
---

## 搭建测试环境

从[Python官网](https://www.python.org)下载[64位Python](https://www.python.org/downloads)并安装，打开CMD窗口输入`pip install selenium`安装web应用程序测试系统，下载[chrome](https://www.google.com/chrome/)安装包，根据安装引导安装chrome浏览器，从[WebDriver for Chrome](http://chromedriver.chromium.org/downloads)网上下载与chrome版本相对应的ChromeDriver并解压到相关位置，将以上安装好的文件的路径添加到系统环境变量中，至此selenium webdriver的运行环境已配置好。

## 识别思路

- 模拟点击切换为滑动验证界面
- 获取验证图片和带有缺口的难图片
- 识别滑动缺口的位置
- 生成滑块拖动的路径
- 模拟拖动滑块拼合并验证
- 若验证失败则重复调用

## 具体步骤

最新版极验出的滑动验证方法没有原始图片了，需要修改页面css样式的属性来获取原始图片。

### 初始化

定义了CrackGeetest类并初始化selenium对象和一些参数，网址是极验的验证码测试页面。

```python
# coding:utf-8

from selenium import webdriver
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from PIL import Image
from io import BytesIO
import time

BORDER = 6


class CrackGeetest():
    def __init__(self):
        self.url = 'https://www.geetest.com/type/'
        self.browser = webdriver.Chrome('F:/chromedriver')
        self.wait = WebDriverWait(self.browser, 10)

    def open(self):
        '''打开网页'''
        self.browser.get(self.url)

    def close(self):
        '''关闭网页'''
        self.browser.close()
        self.browser.quit()
```

### 模拟切换标签

首先模拟点击切换为滑动验证，然后模拟点击弹出验证图片。

```python
    def change_to_slide(self):
        '''切换为滑动认证'''
        label = self.wait.until(
            EC.element_to_be_clickable((
                By.CSS_SELECTOR, '.products-content ul > li:nth-child(2)'
                )))
        return label

    def get_geetest_button(self):
        '''获取初始认证按钮'''
        button = self.wait.until(
            EC.element_to_be_clickable((
                By.CSS_SELECTOR, '.geetest_radar_tip')))
        return button

```

该步骤定义了两个方法，均利用显示等待的方法实现，返回按钮对象，后用click()方法模拟点击。

### 获取验证图片

首先等待验证图片加载完成(wait_pic)并获取整个网页截图(get_screenshot)，然后获取验证图片所在的位置及坐标(get_position)。

```python
    def wait_pic(self):
        '''等待验证图片加载完成'''
        self.wait.until(
            EC.presence_of_element_located((
                By.CSS_SELECTOR, '.geetest_popup_wrap')))

    def get_screenshot(self):
        '''获取整个页面的截图'''
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        # screenshot.show()
        return screenshot

    def get_position(self):
        '''获取验证图片坐标（基于整个页面的截图）'''
        img = self.wait.until(EC.presence_of_element_located((
            By.CLASS_NAME, 'geetest_canvas_img')))
        time.sleep(2)
        location = img.location
        size = img.size
        location['y'] //= 2  # 需调整此值获取有效坐标
        top, bottom = location['y'], location['y'] + size['height']
        left, right = location['x'], location['x'] + size['width']
        return (top, bottom, left, right)
```

再通过返回的位置和坐标，对整个网页截图(get_geetest_image)，最后获取带缺口的验证图片。

```python
    def get_geetest_image(self, name='captcha.png'):
        '''获取验证码图片（根据坐标在整个页面截图上在截图）'''
        top, bottom, left, right = self.get_position()
        print('验证图片坐标', top, bottom, left, right)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop((left, top, right, bottom))
        captcha.save(name)
        return captcha
```

此外还需要获取不带缺口的验证图片，将网页中canvas标签的style删除后，滑块和阴影就不见了，因此可以通过js操作css样式属性，需要用到execute_script()方法，基本上Selenium API没有提供的功能都可以通过执行JavaScript的方式来实现。

执行js脚本后(delete_style)获得了原始的图片，再调用之前的截图方法，就可以获取不带缺口的验证图片。

```python
    def delete_style(self):
        '''执行js脚本，获取完整验证图片'''
        js = 'document.querySelectorAll("canvas")[2].style=""'
        self.browser.execute_script(js)
```

### 识别缺口位置

通过对比得到的两张验证图片来获取缺口位置。

遍历图片中的每个坐标点并获取两张图片的RGB数据，若差距在一定范围内，则认为两个像素相同，若超过一定范围，则说明像素点不同，当前位置即为缺口位置。

函数is_pixel_equal()中定义了一个阈值threshold为60，因为缺口图片中不仅是缺口部分的像素不同，而且其中还设置了一个和缺口大小类似的干扰阴影块，所以需要将范围适当提高，get_gap()方法遍历两张图片的每个像素，再利用is_pixel_equal()方法判断两张图片同一位置的像素。在get_gap()中，left为起始横坐标，即是从滑块的右边开始寻找缺口位置。

```python
    def is_pixel_equal(self, img1, img2, x, y):
        '''判断两个图片的像素是否相同'''
        pix1 = img1.load()[x, y]
        pix2 = img2.load()[x, y]
        threshold = 60
        if abs(pix1[0] - pix2[0]) < threshold \
           and abs(pix1[1] - pix2[1]) < threshold \
           and abs(pix1[2] - pix2[2]) < threshold:
            return True
        else:
            return False

    def get_gap(self, img1, img2):
        '''获取缺口偏移量'''
        left = 60
        for i in range(left, img1.size[0]):
            for j in range(img1.size[1]):
                if not self.is_pixel_equal(img1, img2, i, j):
                    left = i
                    return left
        return left
```

### 计算移动轨迹

滑块的移动轨迹要尽量模拟人移动滑块时的状态，即先加速再减速还可以加入回退和抖动。利用物理学的加速度公式，即可构造出移动轨迹的算法，其中a为滑块的加速度，v为当前速度，v0为初速度，t为时间，s为位移。

`s = v0*t + 1/2*a*t^2`  
`v = v0 + a*t`

先定义变量mid（减速阈值），即加速到什么位置开始减速，在这里前3/5路程加速，后面减速，track返回的是一个列表，其中每个元素代表的是每次移动的距离。

```python
    def get_track(self, distance):
        '''根据偏移量获取移动轨迹'''
        track = []      # 移动轨迹
        current = 0     # 当前位移
        mid = distance * 3 / 5  # 减速阈值
        t = 0.2         # 计算间隔
        v = 0           # 初速度
        distance += 15  # 滑超过过一段距离
        while current < distance:
            if current < mid:
                a = 1   # 加速度为正
            else:
                a = -0.5  # 加速度为负
            v0 = v          # 初速度 v0
            v = v0 + a * t  # 当前速度 v
            s = v0 * t + 1 / 2 * a * t * t  # 移动距离
            current += s    # 当前位移
            track.append(round(s))  # 加入轨迹
        return track
```

### 模拟滑块拼合

获取滑块对象(get_slider)，根据之前得到的移动轨迹拖动滑块(move_to_gap)，然后模拟释放鼠标时的人手抖动(shake_mouse)。

```python
    def get_slider(self):
        '''获取滑块对象'''
        slider = self.wait.until(EC.element_to_be_clickable((
            By.CLASS_NAME, 'geetest_slider_button')))
        return slider

    def shake_mouse(self):
        '''模拟释放鼠标时的抖动'''
        (ActionChains(self.browser).
         move_by_offset(xoffset=-3, yoffset=0).perform())
        (ActionChains(self.browser).
         move_by_offset(xoffset=3, yoffset=0).perform())

    def move_to_gap(self, slider, tracks):
        '''拖动滑块到缺口处'''
        back_tracks = [-1, -1, -2, -2, -3, -2, -2, -1, -1]
        ActionChains(self.browser).click_and_hold(slider).perform()
        for x in tracks:        # 正向
            ActionChains(self.browser).move_by_offset(
                xoffset=x, yoffset=0).perform()
        time.sleep(0.5)
        for x in back_tracks:   # 逆向
            ActionChains(self.browser).move_by_offset(
                xoffset=x, yoffset=0).perform()
        self.shake_mouse()      # 抖动
        time.sleep(0.5)
        ActionChains(self.browser).release().perform()
```

> ActionChains方法列表  
> click(on_element=None) ——单击鼠标左键  
> click_and_hold(on_element=None) ——点击鼠标左键，不松开  
> context_click(on_element=None) ——点击鼠标右键  
> double_click(on_element=None) ——双击鼠标左键  
> drag_and_drop(source, target) ——拖拽到某个元素然后松开  
> move_by_offset(xoffset, yoffset) ——鼠标从当前位置移动到某个坐标  
> move_to_element(to_element) ——鼠标移动到某个元素  
> perform() ——执行链中的所有动作  
> release(on_element=None) ——在某个元素位置松开鼠标左键

执行主体流程，若验证失败，则再次调用crack()进行识别，直至成功。

```python
    def crack(self):
        try:
            self.open()  # 打开网页
            s_button = self.change_to_slide()  # 转换验证方式并点击按钮
            time.sleep(1)
            s_button.click()
            g_button = self.get_geetest_button()
            g_button.click()
            self.wait_pic()  # 图片加载
            # 获取带缺口的验证码图片
            image1 = self.get_geetest_image('captcha1.png')
            self.delete_style()
            image2 = self.get_geetest_image('captcha2.png')
            gap = self.get_gap(image1, image2)
            print('缺口位置', gap)
            gap -= BORDER
            slider = self.get_slider()  # 获取滑块
            track = self.get_track(gap)
            self.move_to_gap(slider, track)
            success = self.wait.until(
                EC.text_to_be_present_in_element((
                    By.CLASS_NAME, 'geetest_success_radar_tip_content'
                ), '验证成功'))
            print(success)
            time.sleep(5)
            self.close()
        except Exception:
            print('Failed & Retry')
            self.crack()


if __name__ == '__main__':
    geetest = CrackGeetest()
    geetest.crack()
```

滑动验证码的识别，基本思路是先用图像识别判断出要滑动的位移，再模拟人拖动滑块的过程（先加速、再减速、适当加入回退和随机抖动），完整代码在GitHub上。
