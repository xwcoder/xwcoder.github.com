---
layout: post
title: 移动web开发—像素
published: true
tags: [像素 px ppi dpi Retina]
---

# 移动web开发—像素

这篇东西主要参考以下文章：

* ppk [《A pixel is not a pixel is not a pixel》](http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html)
* ppk [devicePixelRatio](http://www.quirksmode.org/blog/archives/2012/06/devicepixelrati.html)
* ppk [More about devicePixelRatio](http://www.quirksmode.org/blog/archives/2012/07/more_about_devi.html)
* [支持多种屏幕](https://developer.android.com/guide/practices/screens_support.html#overview)
* 李松峰 [《响应式Web设计》](http://wenku.baidu.com/view/4a5e2e6d783e0912a2162a85.html)
* SCOTT KELLUM [《A Pixel Identity Crisis》](http://alistapart.com/article/a-pixel-identity-crisis)
* 张鑫旭 [《视网膜New iPad与普通分辨率iPad页面的兼容处理》](http://www.zhangxinxu.com/wordpress/2012/10/new-pad-retina-devicepixelratio-css-page/)

熟悉桌面web开发的朋友对于像素(px)都不陌生，在桌面浏览器里我们经常使用px对页面
进行布局， 这对于精确控制页面非常有效，一个<code>font-size: 14px;</code>的文字在各桌面
浏览器上都显示成几乎相同的大小。直观上，一个像素点就是显示器上的一个点，
我们就用这些点来控制页面上的元素，这理解上去如此自然，以至于我们从不用认真思考它。

但是随着移动web的到来，问题开始变的复杂。手机如此小的屏幕却拥有如此高的分辨率，
有的甚至比桌面设备的分辨率还要高，在如此精细的屏幕上如果还用14个像素渲染一个
<code>font-size: 14px;</code>的文字是否还适合阅读？为什么拥有640 * 960的iPhone 4/S在浏览器
里却报告自己是320 * 480的？这时候就有必要梳理一下px相关的概念了。

## 硬件像素/设备像素

> 硬件像素是显示屏上能够显示的最小的点，通常由红、绿、蓝三个子像素构成。从这三个子像素中穿过的光线混合起来，为我们创造了一个像素的颜色。硬件像素与屏幕上的物理元素一一对应，不能拉伸、扭曲，也不能再分。这些特点让硬件像素很像原子——任何设计作品中最基本的单位。

这里有必要先区分下物理长度单位和逻辑长度单位。我们在真实世界里使用的米、厘米、英寸等属于物理长度单位。
而px则是逻辑长度单位，1px在物理度量上有多大取决于屏幕的分辨率和和屏幕的尺寸，即屏幕的像素密度PPI。

### PPI和DPI

PPI是英文Pixels Per Inch的缩写，由字面意思很好理解：每英寸的像素数量，即像素密度。计算公式也很简单：

![ppi](/imgs/ppi-formular.jpg)

(X：长度像素数；Y：宽度像素数；Z：屏幕尺寸即对角线长度)

那么以毫米mm为单位一个像素的大小就是<code>1px=25.4/PPI(1in=25.4mm)</code>。iPhone 4/S的PPI是326，则像素大小为<code>25.4/326 ~= 0.078mm</code>。

<img width="100%" src="/imgs/px-size.jpeg" alt="">

对于非Retina屏的桌面设备，像素密度通常在72-120之间。

> 说一个题外，一般桌面浏览器的默认字体大小是16px，即1em=16px。在印刷领域，m是英文中最宽的字母，所以用em来表示印刷的字宽，标准是12pt。
1pt=1/72in，window桌面显示设备的默认PPI是96，即1px=1/96in，简单换算1em=16px。不同的设备默认字体大小会有例外，比如kindle touch
的默认字体大小是26px，即1em=26px。

DPI(Dots Per Inch)是印刷行业使用的概念，表示打印机每英寸可以喷墨的点数。PPI的概念就是借鉴自DPI。在显示器上通常可以混用这两个概念。

### Retina屏

苹果公司在发布iPhone 4时引入了视网膜屏(Retina display)这个概念。之所以叫做视网膜屏是因为屏幕的PPI太高，高到
人眼视网膜分辨不出屏幕上的像素点。同样为3.5寸屏幕，iphone 4/S的PPI为326，是iPhone 3G/S的两倍。

<img width="100%" src="/imgs/retina-01.png" alt="">

第三代iPad发布会上，苹果给出了Retina设计标准的公式：

![retina](/imgs/retina-formular.jpg)

> 其中 a代表人眼视角，h 代表像素间距，d代表肉眼与屏幕的距离。符合以上条件的屏幕可以使肉眼看不见单个物理像素点。这样的IPS屏幕就可被苹果称作“Retina显示屏”。将通常使用距离代入上公式可知：行动电话显示器的像素密度达到或高于300ppi就不会再出现颗粒感；手持平板类电器显示器的像素密度达到或高于260ppi就不会再出现颗粒感，而苹果电脑的Retina显示器像素密度只要超过200ppi就无法区分出单独的像素。

Retina是苹果注册的命名，其他厂商只能使用类似“HI-DPI”等名称，不过意思是一样的。

## 参照像素

既然px不是物理单位，而PPI越高px尺寸越小，在各厂商都在追求高PPI的今天1px到底要多大才适合人类阅读呢。w3c为所有基于像素
的度量定义了一个标准，叫做[参考像素](http://www.w3.org/TR/CSS2/syndata.html#length-units)。

<img width="100%" src="/imgs/px-01.jpeg" alt="">

对于手机来说0.148mm是一个比较合理的大小。

参照像素是与硬件像素无关的，这样基于像素的设计就不必局限在硬件像素上。同样的参照像素，在手机上就会很小，而在投影设备上就会很大。

## 设备独立像素/密度无关像素

不同的设备可能会有不同的PPI，1个设备像素代表的物理长度也会不一样，这就为应用程序的开发适配带来了困难。
为解决这个问题，操作系统引入了一层抽象概念：[设备设备独立像素/密度无关像素(device-independent pixel/density-independent pixel, dip/dp)](https://developer.android.com/guide/practices/screens_support.html#overview)

Android中使用DIP/DP，ios中使用PT。开发应用程式时就可以使用DP进行布局和度量，然后由系统转换为设备像素进行渲染。
例如一个44 * 44dp的元素，在非Retina屏中等于44 * 44物理像素，在Retina屏(device-pixel-ratio=2的情况)中等于88 * 88物理像素。

<img width="100%" src="/imgs/dp-01.png" alt="">
<img width="100%" src="/imgs/ios-04.png" alt="">

在浏览器中怎么获取dip呢？

ppk在12年的测试结果是：无法准确获取。ios中<code>screen.width/height</code>给出的是dip，而部分Android设备<code>screen.width/height</code>给出的是设备像素。

目前(2017-02)我的测试结果是：ios和Android设备中<code>screen.width/height</code>给出的都是dip。<em>这个结论并不可靠，因为我的测试并没有覆盖足够的Android设备。</em>

## device-pixel-ratio

> window.devicePixelRatio是设备硬件像素和设备独立像素的比值。公式：window.devicePixelRatio=硬件像素/dip。

- iPhone 3G/S的devicePixelRatio是1
- iPhone 4/S、iPhone 5/C/S、iPhone 6的devicePixelRatio是2
- iPhone 6 Plus的devicePixelRatio是3
- iPad 1 & 2、iPad Mini 1的devicePixelRatio是1
- iPad 3 & 4 / Air和iPad Mini 2的devicePixelRatio是2

更多[设备信息](https://material.io/devices/)

不同于其他iPhone，iPhone 6 Plus在显示上有一点特别，它会对元素进行向下采样。

iPhone 6 Plus的devicePixelRatio是3, dips是414 x 736px，理论上设备分辨率应该是1242 x 2208px, 
但是iPhone 6 Plus的硬件像素只有1080 x 1920px，因为其会对内容进行向下采样显示。

<img width="100%" src="/imgs/ios-01-2.png" alt="">

这样在计算上会有半个像素的情况，造成显示上出现毛刺，但是iPhone 6 Plus的PPI足够高，如果不是在非常近的距离观看是察觉不到这些小瑕疵的。

相较于ios设备，Android设备的情况要复杂的多，因为Android是开源系统，OEM厂商可以对像素进行“随心所欲”的处理。比如，最早的Galaxy Tab
和Kindle Fire的屏幕尺寸和分辨率是一样的，但Galaxy Tab根据参照像素进行了调整，每个像素是Kindle Fire的1.5倍，Fire使用的是硬件像素。
开发人员可能会以为屏幕一样大，而且浏览器内核都是WebKit，那么网站呈现就会一样。但是事实并非如此，对于网站来说，Galaxy Tab是400px * 683px，而Kindle Fire是600px * 1024px。

<img width="100%" src="/imgs/standard.jpg" alt="">

## css像素

类似于设备独立像素，css像素是在web 开发中的一个抽象的概念，它与屏幕无关，css像素与设备像素可能不是一一对应的。
在多数非Retina屏的桌面设备上，“正常状态”下一个css像素对应一个硬件像素。这里说的“正常状态”在桌面浏览器上是指缩放为100%。

在一个缩放100%的页面上放置一个128px * 128px的元素，然后将页面放大到200%，元素的大小变成了以前的4倍，占据256px * 256px的大小，
但是在css定义上元素还是128px * 128px, 这时一个css像素实际上等于两个硬件像素。

通过图示来解释这个过程。

100%时，一个css像素等于一个硬件像素，css像素与硬件像素重叠。

![100%](/imgs/csspixels_100.gif)    

现在缩小页面，css像素开始收缩，一个硬件像素覆盖了多个css像素。

![out](/imgs/csspixels_out.gif)    

对页面进行放大，css像素开始变大，一个css像素覆盖了多个硬件像素。

![in](/imgs/csspixels_in.gif)    

默认情况下，在高清设备上，一个css像素可能会对应多个硬件像素，或者表述为用多个硬件像素显示一个css像素。
这一点对于web开发和设计来说非常重要，否则开发和设计工作就会变得复杂许多。

假设iPhone 3G/S和iPhone 4/S上都是一个css像素对应一个硬件像素，iPhone 3G/S上1px=0.156mm，
而在iPhone 4/S上这个值减小了一半，只有0.078mm，同样一个20px * 20px的按钮，渲染在iPhone 4/S上
只有iphone 3G/S上的1/4大小。也就是说PPI越高，文字和按钮也越小越难看清。我们就不得不针对iPhone 4/S对字体大小进行调整。

为了解决这个问题，这些高清设备都会将自己的分辨率“谎报”成更小的值，像文章开头提到的iPhone 4/S的硬件分辨率
是640 * 960，但是报告给浏览器的却是320 * 640，也就是说一个css像素对应两个硬件像素。

在开发时，依然像平时一样通过css像素设置元素大小，<code>.btn{width: 20px; height: 20px}</code>，iPhone 4/S在背后会用
40 * 40个硬件像素对其进行渲染，然后我们看到的按钮就是正常大小。
对同样一段文本使用css定义<code>.txt{font-size:16px;}</code>，在iPhone 3G/S和iPhone 4/S上看到的字体大小也是一样的。

<img width="100%" src="/imgs/px-px.jpeg" alt="">

前文在讲“参照像素”时讲过在手机上一个像素的大小为0.148mm是比较适合阅读的。

<img width="100%" src="/imgs/px-02.jpeg" alt="">

<img width="100%" src="/imgs/px-03.jpeg" alt="">

为了达到适合阅读的目的，不同PPI的设备都会参考“参照像素”对css像素进行调整，这样硬件像素和css像素之间就有一定的比例关系。iPhone 4/S的硬件像素(640 * 960)
与css像素(320 * 480)的比值是2，即一个css像素的大小为0.156mm(2个硬件像素)，这是比较理想的适合阅读的尺寸。

web开发人员使用css像素进行设计，系统自动将css像素转换为设备像素进行渲染。

### 网页中的位图显示

位图是由图像像素组成的，每个图像像素都有自己特定的位置和颜色等信息。单位长度内图像像素越多，图像质量越高。

当使用img标签在网页中插入一张位图时，图片的大小是用css像素控制的。
在非Retina屏幕上100%显示这张图片，一个图像像素对应一个硬件像素，即一个硬件像素渲染一个图像像素，图片就能保真显示。
但是在Retina屏中，就会用多个硬件像素渲染一个图像像素，例如iPhone 4/S中就会用4个硬件像素点渲染一个图像像素，一个图像像素分成4份，颜色只能
取近似值，于是图片就模糊了。

<img src="/imgs/bitmap-pixels.png" alt="">

要想图片清晰显示，同时保持在非Retina屏中的视觉尺寸，可以将一张4倍大小的图片缩放到原来的1/4。如果原始图片尺寸是200 * 200px，那么
就需要一张400 * 400px的图片，同时css定义仍然是<code>img{width: 200px; height: 200px;}</code>。这样一个位图像素就由一个硬件像素来渲染了。
但是在非Retina屏中，图片被压缩，造成图像像素采样上的浪费。

<img src="/imgs/downsampling.png" alt="">

网页中对于前端渲染的部分可以通过window.devicePixelRatio判断需要加载的图片。对于背景图图片可以使用媒体查询，通过<code>background-size</code>控制。
例如：
<pre>
.bg {
    background: url(images/bg.png) 0 0 no-repeat;
    background-size: 837px 455px;
}
@media (-webkit-min-device-pixel-ratio: 2) {
    .bg {
        background-image: url(images/bg@2x.png);
    }
}
</pre>

## 总结

在web开中主要用到的是硬件像素和css像素，css像素和硬件像素并不总是一一对应的。

PPI值越高px的尺寸越小，w3c给出了标准[参照像素](http://www.w3.org/TR/CSS2/syndata.html#length-units)。通常Retina设备
都会参考“参照像素”对css像素进行调整，这时css像素和硬件像素就不是一一对应的。
