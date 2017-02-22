# 移动web开发 - 适配

需要了解`css像素`，`viewport`等相关概念。可参考[移动web开发-像素](https://github.com/xwcoder/xwcoder.github.com/issues/8)、[移动web开发-viewport](https://github.com/xwcoder/xwcoder.github.com/issues/12)。

## 适配什么

设计师通常会按一个固定宽度出设计稿，目前常用的是750px(iPhone 6的物理屏幕宽度)，而页面最终要呈现在各种尺寸的屏幕上。因此适配就是要使页面元素在不同尺寸的设备上能够自动缩放，而且缩放最好是等比的，从而看上去大致一样。例如设计稿上某个按钮占整个宽度的80%，那么在不同设备上此按钮总是占屏幕宽度的80%，同时还要考虑保持按钮本身的宽高比。

一般我们关注的内容包括：
- 盒模型（元素宽高、padding、margin）
- 字体
- 图片

对于`盒模型`，我们希望是等比缩放的，当然根据具体产品需求也许会要求最大最小值。

对于`字体`，通常我们并不需要等比缩放，因为我们希望字体在一个合适阅读的大小，而且在越大的屏幕上显示越多的内容。对于标题/slogan等也许会需要等比缩放。一切看产品需求。

对于`图片`，希望保真显示，并且对于位图可以按需加载不同的图片，以节省流量。可以使用的技术有`media query`, `background-size`, `image-set`, `max-width/max-height`, `js`等。不展开。

## 适配方案

做适配时我们可用的技术有：`px`, `media query`, `百分比`, `Flex`, `viewport缩放`, `rem`, `vw/vh`等。

### 宽度自适应

1、设置`ideal viewport`

```html
<meta name="viewport" content="width=device-width,initial-scale=1">
```

2、 宽度自适应：使用`百分比`，`Flex`。此时需要对照设计稿尺寸计算元素宽度的百分比。根据需求有时某些元素会需要`定值(px)`，此时如果设计稿是按大尺寸屏幕设计的，那么页面在小屏幕上有可能会出现错乱。

3、高度使用定值或者伪元素等方案保持宽高比。

高度可以使用定值，例如很多新闻站点的文字新闻列表。

也可以使用伪元素等方案使元素保持宽高比：

```css
.item:before {
    content: "";
    display: block;
    padding-top: 56.25%;
}
```

4、使用`media query`进行补充调整。

[手机搜狐网](https://m.sohu.com/)、[腾讯](http://xw.qq.com/index.htm)。

### 固定页面和`layout viewport`宽度, `visual viewport`缩放

将`layout viewport`的值设置成固定值。便于开发，最好设置成设计稿的宽度，例如640px。

```html
<meta name="viewport" content="width=640">
```

这样就可以像PC端切固定宽度的页面一样，根据设计稿标注/测量值设置元素的尺寸。然后`visual viewport`会自动根据`layout viewport`缩放。

> 没有设置`initial-scale`时，`visual viewport`的初始宽度等于`layout viewport`的宽度。

因为Android 4.4之前版本的缩放bug，需要再加上`target-densitydpi=device-dpi`：

```html
  <meta name="viewport" content="target-densitydpi=device-dpi,width=640">
```

对于开发来说，这种方式简洁高效。但是在物理分辨率低的设备上会有失真的情况，好在硬件升级速度非常快，这种情况显得不那么重要了。

[荔枝FM](http://m.lizhi.fm/)、[网易金币商城](http://c.m.163.com/CreditMarket/default.html)。

### 使用rem

> "rem unit": Equal to the computed value of font-size on the root element. - [w3c](https://www.w3.org/TR/css3-values/#rem)

因为`rem`是相对值，所以在布局时可以使用`rem`，然后根据`layout viewport`的宽度动态设置`<html>`元素的`font-size`，从而达到适配目的。设置`<html>`元素的`font-size`有两种方式。

#### 方式一：使用media query

如：

```css
@media (min-device-width : 375px) and (max-device-width : 667px) and (-webkit-min-device-pixel-ratio : 2){
      html{font-size: 37.5px;}
}
```

这种方式需要控制好断点的粒度，而且不够精确，所以不常用。

#### 方式二：使用js，根据设计稿宽度和`layout viewport`宽度计算

假设设计稿的宽度是750。

设定`layout viewport`宽度为750px(设计稿的宽度)时，`1rem=100px`，即`<html>`元素的`font-size`大小是100px。这只是个基准值，理论上可以设定为任意大小，之所以设定成100px是为了方便计算。然后根据页面实际的`layout viewport`宽度计算并设置`<html>`的`font-size`。

```js
(function (doc, win) {

   var docEl = doc.documentElement

   function refreshRem () {
     docEl.style.fontSize = docEl.clientWidth / 750 * 100 + 'px'
   }

   refreshRem()

})(document, window);

```

设置css盒模型的大小时，只需使用设计稿上标注/测量的长度除以100即可。例如对于设计稿上一个尺寸为`180 x 180`的元素，设置成`1.8rem x 1.8rem`。

```css
.item {
  width: 1.8rem;
  height: 1.8rem;
}
```

如果项目中使用sass等css预处理工具，也可以使用函数将`px`转换为`rem`。

```css
@function px2rem($px){
    $rem : 100px;
    @return ($px/$rem) + rem;
}

.item {
  width: px2rem(180px);
  height: px2rem(180px);
}
```

还有一种设置基准值的方法：将设计稿等分。例如10等分，那么对于设计稿来说`1rem = 750px / 10 = 75px`，即基准值是`75px`，在实际页面中一样将`layout viewport`10等分：

```js
(function (doc, win) {

    var docEl = doc.documentElement

    function refreshRem () {
        docEl.style.fontSize = docEl.clientWidth / 10 + 'px'
    }

    refreshRem()

})(document, window);
```

将`px`转换为`rem`时，使用设计稿标注/测试值除以`75px`。

```js
@function px2rem($px){
    $rem : 75px;
    @return ($px/$rem) + rem;
}
```

当然，转换工作也可以使用gulp、grunt、webpack等任务/构建工具完成。

由于`rem`的大小是根据`layout viewport`的宽度计算的，所以当页面`resize`，`pageshow`时需要重新计算。

```js
var timer

window.addEventListener('resize', function () {
  clearTimeout(timer)
  timer = setTimeout(refreshRem, 300);
}, false)

window.addEventListener('pageshow', function (e) {
  if (e.persisted) {
    clearTimeout(timer)
    timer = setTimeout(refreshRem, 300);
  }
}, false)
```

[淘宝](https://m.taobao.com/)

#### 字体

通常情况下段落字体仍然使用`px`。我们希望文本的`font-size`在一个合适阅读的大小，然后在更大的屏幕上显示更多的内容，而不是随着屏幕的曾大而等比放大。当然，特殊需求除外。

### 使用`vw/vh`

> Length units representing 1% of the viewport size for viewport width (vw), height (vh)。

目前[兼容性](http://caniuse.com/#search=vw)稍差。可根据项目目标设备的情况选用。
