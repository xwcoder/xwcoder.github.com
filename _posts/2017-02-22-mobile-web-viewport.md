# 移动web开发 - viewport

这篇东西主要整理自ppk的系列文章：
- ppk [A tale of two viewports — part one](http://www.quirksmode.org/mobile/viewports.html)
- ppk [A tale of two viewports — part two](A tale of two viewports — part two)
- ppk [Meta viewport](http://www.quirksmode.org/mobile/metaviewport/)
- ppk [A new Device Adaptation spec](http://www.quirksmode.org/blog/archives/2015/09/a_new_device_ad.html)
- ppk [Mobile - Table of contents](http://www.quirksmode.org/mobile/)
- ppk [screen.width is useless](http://www.quirksmode.org/blog/archives/2013/11/screenwidth_is.html)
- w3c [CSS Device Adaptation Module](https://drafts.csswg.org/css-device-adapt/)
- Apple [Configuring the Viewport](https://developer.apple.com/library/content/documentation/AppleApplications/Reference/SafariWebContent/UsingtheViewport/UsingtheViewport.html)
- Apple [Supported Meta Tags](https://developer.apple.com/library/content/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html#//apple_ref/doc/uid/TP40008193)
- [Support for target-densitydpi is removed from WebKit](http://stackoverflow.com/questions/11592015/support-for-target-densitydpi-is-removed-from-webkit)
- [Bug 88047 - Remove support for target-densitydpi in the viewport meta tag](https://bugs.webkit.org/show_bug.cgi?id=88047)
- [Customize Android Browser Scaling with target-densityDpi](http://darkforge.blogspot.com/2010/05/customize-android-browser-scaling-with.html)
- [viewport - target-densitydpi](https://docs.google.com/presentation/d/1rmxwWa9P6_xHqonmh5ONXRS-jPc5XKbnv99Rjkhe04s/present?slide=id.i30)

需要了解`设备像素`、`设备独立像素(dip)`、`css像素`等概念。可以参考[移动web开发 - 像素](https://github.com/xwcoder/xwcoder.github.com/issues/8)。

由于移动端的概念相对复杂，所以先从桌面浏览器相关概念开始介绍。

## 桌面浏览器部分

首先看几组web开发中会接触到的尺寸度量值。

### Screen size

![desktop_screen.jpg](http://xwcoder.github.io/imgs/20170222/287844C1BB6BB891F4ED208AF95CC585.jpg)

严格来说`screen.width/height`取到的值并不可靠。

对于桌面浏览器，Chrome和Safari中`screen.width/height`存储的是以物理像素或者设备独立像素为单位的屏幕尺寸。这取决于是不是Retina屏幕。比如13寸Retina屏的MacBook pro，`dpr=2`，物理分辨率是`2560 x 1600`，而`screen.width = 1280 screen.height = 800`，而且会随着手动调整分辨率而变化。Edge/IE和Firefox中`screen.width/height`的值还会随着页面的缩放而变化。

在移动端浏览中，`screen.width/height`通常存储的是`ideal layout viewport`的尺寸。在一些旧的移动端浏览器中`screen.width/height`存储的是设备的物理分辨率。

> 关于`ideal viewport`在移动端的部分会有介绍。

所以结论是：我们没有办法准确得到屏幕的尺寸。

web开发中通常不太需要关心`screen.width/height`，除了统计需求只在极少情况下会需要用到这组值，比如定位桌面程序的悬浮窗。

![4FAB31F4-65D7-45AB-B264-872DE757ACE9.png](http://xwcoder.github.io/imgs/20170222/8780D9236827CD4065DFEAC520A3FF4D.jpg)

### Window size

![desktop_inner.jpg](http://xwcoder.github.io/imgs/20170222/F6A4504C63782771966AED14BCD6D7AB.jpg)

`window.innerWidth/Height`获取的是浏览器可见窗口(browser window)的大小，包含滚动条的宽度，不包含工具栏、任务栏、地址栏等。它代表了我们在屏幕中能看到多少网页内容。

<span style="color:#ff0000;">`window.innerWidth/Height`的度量单位是`css像素`。</span>所以当缩放页面时这组值会相应变化。

### Scrolling offset

![desktop_page.jpg](http://xwcoder.github.io/imgs/20170222/736F31CAA4087EBDCC4348E62D0DA899.jpg)

`window.pageXOffset/window.pageYOffset`代表网页水平和垂直滚动的距离。度量单位是`css像素`。

### viewport

> viewport就是浏览器视口，它等于浏览器的可见窗口(borwser window)

> The viewport, in turn, is exactly equal to the browser window: it’s been defined as such. 

> In CSS 2.1 a viewport is a feature of a user agent for continuous media and used to establish the initial containing block for continuous media

- viewport并不是HTML中的元素。
- 它只是拥有browser window的宽高(桌面浏览器)。
- 它的度量单位是`css像素`。
- 它限制了`<html>`元素的默认宽度。

我们知道默认情况下大多数块级元素的宽度是父元素宽度的100%，那么作为文档顶级元素的`<html>`的默认宽度是多少呢？

在默认情况下`<html>`元素的宽度就是`viewport`的100%。

#### viewport的大小

![desktop_client.jpg](http://xwcoder.github.io/imgs/20170222/DF7B847651D50D6C6E1208F51B6089DC.jpg)

可以通过`document.documentElement.clientWidth/Height`获取viewport的大小。

因为桌面浏览器只有一个viewport，viewport就是浏览器的可见窗口，所以`document.documentElement.clientWidth/Height`近似等于`window.innerWidth/Height`，差别是`document.documentElement.clientWidth/Height`不包含滚动条的宽度。

> 浏览器中之所以同时存在`document.documentElement.clientWidth/Height`和`window.innerWidth/Height`这两组值，是由于当年的浏览器大战：Netscape只支持`window.innerWidth/Height`，IE只支持`document.documentElement.clientWidth/Height`，因此所有的浏览器都开始支持clientWidth/Height，但是IE并没有支持`window.innerWidth/Height`。

> 还有一点需要特别注意：`document.documentElement.clientWidth/Height`给出的永远是viewport的大小，而不是`<html>`元素的大小，这和其他元素是不一样的。我们可以通过css设置`<html>`的大小。

![desktop_client_smallpage.jpg](http://xwcoder.github.io/imgs/20170222/4A60BE1E789E2517407A6B907B8010A3.jpg)

当我们放大页面时，css像素会变大，viewport可容纳的css像素数会变化，即viewport的尺寸会变小。反之亦然。

### 获取`<html>`元素的尺寸

通过`document.documentElement.offsetWidth`获取`<html>`元素的大小。

![desktop_offset.jpg](http://xwcoder.github.io/imgs/20170222/FA83DB965A9CC3F938905A53BA69ED29.jpg)

![desktop_offset_smallpage.jpg](http://xwcoder.github.io/imgs/20170222/231844ADE4C39FDB6BE0867E4D9A98C1.jpg)

### 获取文档的宽度(document width)

目前没办法直接获取文档的宽高。有一个间接办法：计算每个独立块元素的宽高、margin、padding，然后相加。但是也并不可靠。

### 事件中的坐标

通常在鼠标事件中有三组坐标值是我们需要的：`pageX/Y`、`clientX/Y`、`screenX/Y`。
- `pageX/Y`相对于`<html>`元素的坐标，以`css像素`为单位。
- `clientX/Y`相对于`viewport`的坐标，以`css像素`为单位。
- `screenX/Y`相对于屏幕的坐标，单位值参考`Screen size`部分。

---

## 移动端部分

现在来看看移动端的情况。由于大多时候我们并不关心高度，所以以下讨论除非特别说明都是指宽度。

### layout viewport和visual viewport

智能手机问世之初，绝大部分网站并没有为移动设备进行显示优化。这时手机厂商就面临一个问题：如何在移动设备如此小的屏幕上显示pc网页。浏览器引入了两个viewport（`layout viewport`和`visual viewport`）来解决这个问题。

`layout viewport`用于css布局，度量单位是`css像素`，`<html>`默认宽度是`layout viewport`宽度的100%。默认情况下`layout viewport`是多大呢？ Safari iPhone 是980px, Android WebKit是800px, IE是974px(这个值会随着厂商的修改而变化)。这样网页就在一个比较大的viewport上进行布局，即使没有针对移动端进行优化的网页也可以正常布局而不会出现错乱。

但是这时网页上的元素和文字就会变得非常小，难以阅读（由于css像素和设备像素的对应关系）。用户可以通过pinch-out放大页面来”看清“页面上的内容，试想一下，如果此时浏览器的行为像桌面端一样，`layout viewport`就会变小，不但会造成页面的reflow，极大可能页面布局会乱掉（取决于页面的布局方式）。

浏览器通过引入`visual viewport`来解决这个问题，`visual viewport`是用户观看网页的窗口，当用户在pinch-in/pinch-out时，`layout viewport`是不变的（这样页面就不会reflow），进行缩放的是`visual viewport`，`visual viewport`也是以`css像素`进行度量的。

> 可以这样来理解`layout viewport`和`visual viewport`，将`layout viewport`想象成一张画布，大小是不变的，网页就是画布上的内容，我们用手机对画布进行拍照，我们可以通过调整相机焦距来选择是拍画布的全部内容还是部分内容，调整焦距时显示在相机里的画布内容会放大/缩小，但是画布本身是不变的，这时相机就相当于`visual viewport`。

![mobile_visualviewport.jpg](http://xwcoder.github.io/imgs/20170222/3977700249B2F7900591ADCEEE011092.jpg)

还有一点需要指出，通常浏览器都会选择这样的行为：当缩放到最小时，`visual viewport`的宽度刚好等于`layout viewport`的宽度，这时我们通过`visual viewport`可以看到页面的全部(宽度上)。<span style="color:#ff0000;">这也是浏览器初次加载页面时的默认行为</span>，这个默认行为非常有用，它是之后我们要讲的一种布局方式的基础。

![mobile_viewportzoomedout.jpg](http://xwcoder.github.io/imgs/20170222/B841ADD242207CFB90EB666076CF914F.jpg)

### 获取`layout viewport`和`visual viewport`的大小

通过`document.documentElement.clientWidth/Height`获取`layout viewport`的大小。

通过`window.innerWidth/Height`获取`visual viewport`的大小。

### 其他尺寸值

Scrolling offset和`<html>`元素的尺寸与在PC端获取方式一致。

---

最后看看`meta viewport tag`。

### meta viewport tag

我们可以通过`meta viewport tag`来设置`layout viewport`的宽度。最初苹果公司引入了`meta viewport tag`用于设置viewport和缩放，之后其他系统也对此进行了跟进支持。

`meta viewport tag`语法如下：

```html
<meta name="viewport" content="name=value, name=value">
```

例如：

```html
<!--将layoutviewport的宽度设置为640px-->
<meta name="viewport" content="width=640"> 
```

`meta viewport tag`支持如下6个指令：
- `width`：设置layout viewport的宽度。
- `initial-scale`：设置初始缩放系数和layout viewport的宽度。
- `minimum-scale`：设置最小缩放系数。
- `maximum-scale`: 设置最大缩放系数。
- `height`：设置layout viewport的高度，浏览器并不支持，通常我们也不需要设置这个值。
- `user-scalable`：设置是否允许用户进行缩放，`no`：不允许。

### ideal viewport（最佳视口/完美视口）

既然我们可以设置`layout viewport`的大小，那么对于网页来说哪个尺寸才是最佳尺寸呢？答案是：设备不同，最佳尺寸也不同。例如iPhone 3g/s的最佳尺寸是320 x 480(物理像素是320 x 480)，iPhone 4/s是320 x 480（物理像素是640 x 960），iPhone 6是375 x 667(物理像素是750 x 1334)。更多设备参考[这里](http://www.quirksmode.org/mobile/metaviewport/#t00)。

我们将设置成最佳尺寸的`layout viewport`称为`ideal viewport`。

大部分设备的`ideal viewport`尺寸等于以dip度量的屏幕尺寸，[参考](https://material.io/devices/)

#### 设置ideal viewport

通过`width=device-width`或者`initial-scale=1`都可以将`layout viewport`设置成`ideal viewport`的值。

有一点需要特别注意：__所有`scale`相关的指令都是相对于`ideal viewport`的值进行计算的__。

#### 获取ideal viewport的值

通过`meta viewport tag`设置`ideal viewport`，然后通过`document.documentElement.clientWidth/Height`获取。

```html
<meta name="viewport" content="width=device-width,initial-scale=1">
```

```js
var w = document.documentElement.clientWidth;
var h = document.documentElement.clientHeight;
```

### 缩放公式

```js
visual viewport width = ideal viewport width / zoom factor
zoom factor = ideal viewport width / visual viewport width
```

缩放系数总是相对于`ideal viewport`的，与当前`layout viewport`的大小无关。

### initial-scale

前边说`initial-scale`用来设置初始缩放系数和layout viewport的宽度。现在看看当设置`initial-scale`时究竟发生了什么。

当设置`initial-scale`时发生了如下两件事：

1. 设置页面的缩放系数（zoom factor）, 根据缩放系数计算出`visual viewport`的宽度。
2. 设置`layout viewport`的宽度等于`visual viewport`的宽度。

> bugs:
1. 部分Android设备不支持`1`以外的值。

### zoom factor的取值范围

有一条约束：__`visual viewport`的宽度不能大于`layout viewport`的宽度。__

关于`minimum-scale`和`maximum-scale`取值范围，Apple的文档表述如下：

> minimum-scale: The default is 0.25. The range is from > 0 to 10.0.
maximum-scale: The default is 5.0. The range is from >0 to 10.0.

不同系统支持情况不同，[详见](http://www.quirksmode.org/mobile/metaviewport/#t40)。

### width与initial-scale的冲突

既然`width`和`initial-scale`都可以设置`layout viewport`的宽度，那么同时设置这两个值时就会产生冲突。例如：

```html
<meta name="viewport" content="initial-scale=1, width=400">
```

对于iPhone 4/s来说:
- `initial-scale=1`：在竖屏下`layout viewport`的宽度是320px，横屏下是480px。
- `width=400`：横竖屏下`layout viewport`的宽度都是400px。

浏览器处理这种冲突的方案是：__取最大值__。竖屏下`layout viewport`的宽度是400px，横屏下是480px。

Android WebKit在处理这种情况时有兼容问题，对于Android 4.4之前的Android WebKit，如果宽度等于`device-width`或者小于320，总是使用`ideal viewport`的宽度；如果宽度大于320，总是使用`width`制定的值。

### `target-densitydpi`

Android 4.4之前的Android WebKit对`width`指令和缩放支持非常糟糕。

> 以三星 Note II 为例，它的`device-width`是 360px。如果设置meta viewport tag的 width为小于等于 360 的值，则不会有任何作用；而设置为大于 360 的值，不会使画面产生缩放，而是出现了横向滚动条。

可以使用`target-densitydpi`来解决缩放问题。`target-densitydpi`是Android的私有属性，在Android 4.4之后已声明被废弃。

`target-densitydpi`的作用是设置目标设备的像素密度等级，它可以取如下值：
- `device-dpi`：使用设备自身的dpi。
- `low-dpi`：120dpi。
- `medium-dpi`：160dpi，默认值。
- `high-dpi`：240dpi。
- `<value>`： 指定一个具体的dpi，取值范围在70–400之间。

```html
<meta name="viewport" content="target-densitydpi=device-dpi, width=400">
```

> In the "viewport" meta tag, you can specify "target-densityDpi".
If it is not specified, it uses the default, 160dpi as of today.
Then the 1.0 scale factor specified in the viewport tag means 100%
on G1 and 150% on Sholes. If you set "target-densityDpi" to
"device-dpi", then the 1.0 scale factor means 100% on both G1 and Sholes.

当设置成`target-densitydpi=device-dpi`可以正确的缩放。

__尽量避免使用target-densitydpi。__

---

## 总结

移动浏览器中有两个viewport：`layout viewport`, `visual viewport`。`layout viewport`用于网页布局。我们将设置成最佳尺寸的`layout viewport`称为`ideal viewport`。

可以通过`meta viewport tag`设置缩放系数和`layout viewport`的宽度。

通过设置`width=device-width`或者`initila-scale=1`都可以将`layout viewport`设置成最佳宽度。

__大部分设备的`ideal viewport`尺寸等于以dip度量的屏幕尺寸，__[参考](https://material.io/devices/)

`visual viewport`的宽度不会大于`layout viewport`的宽度。

缩放(scale)相关的指令都是相对于`ideal viewport`的宽度进行计算。

缩放系数公式：
`visual viewport width = ideal viewport width / zoom factor`    
`zoom factor = ideal viewport width / visual viewport width`

当设置`initial-scale`时浏览器做了如下工作：
1. 根据`initial-scale`的值计算`visual viewport`的宽度。
2. 设置`layout viewport`的宽度等于`visual viewport`的宽度。

没有设置`initial-scale`时，`visual viewport`的初始宽度等于`layout viewport`的宽度。

当没有通过`width`或者`initial-scale`指令设置`layout viewport`的宽度时，使用浏览器的默认宽度。

通过`document.documentElement.clientWidth/Height`可以获取`layout viewport`的大小。
通过`window.innerWidth/Height`可以获取`visual viewport`的大小。

bugs:
1. 部分Android设置不支持设置`1`以外的`initial-scale`。
2. Android 4.4之前的Android WebKit存在缩放问题，可以通过私有属性`target-densitydpi=device-dpi`解决。
#
