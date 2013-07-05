---
layout: post
title: 输入法概念
published: true
tags: [IMF 输入法 输入法概念 IME]
---

## 写在前面 ##
输入法只有中日韩这种象形文字语系才需要(CJK only), 西方语系的文字都可以通过键盘直
接输入。

## 输入法相关概念 ##
在现代，输入法从上层到底层包括三个方面：

1. 输入法界面
2. 输入法引擎
3. 输入法框架

### 输入法界面
我们使用搜狗拼音时看到的选词界面、设置界面就是输入法界面。输入法界面分为两部分：

* 输入面板：选词界面
* 设置界面：如搜狗的设置界面

输入法框架通常会提供一个输入界面。但输入法界面通常与输入法框架本身联系并不紧密，
所以可以由外部程序来提供。

### 输入法引擎
输入法引擎主要就是用来处理用户输入并给出相应候选词的程序。拿一般的拼音输入法举例，
用户输入「xian」，要给出怎样的候选词呢？「先」、「现」、还是「西安」？当然也可以
都给出，这取决于输入法引擎怎么分解/理解输入了。当然整句的输入处理会更加复杂。

### 输入法框架
输入法框架主要处理输入法引擎，输出界面，系统桌面环境，各应用程序之间的通讯。

举例来说，QQ聊天时使用拼音输入法输入的整个过程。QQ对话框中提供了一个文本框，用来
显示用户输入的字词。像QQ文本框中输入一句话'我很开心'的过程如下：

1. 鼠标点击QQ对话框中的文本框，使文本框获得焦点。
2. 按下按键'wohenkaixin'。
3. 输入法引擎获得按键信息，用户都按了哪些键，本例是'wohenkaixin'。
4. 输入法引擎经过分析处理，给出'wohenkaixin'对应的中文可能是'我很开心'。
5. 输入法通过界面在光标的位置显示输入法引擎给出的候选词'我很开心'。
6. 用户按相应数字键(不同输入法选择按键可能不同)选择自己想要的文字。
7. 将选中的文字传递给QQ对话框，QQ对话框中显示输入的文字。

对于所有的输入法来说，这个过程中的部分功能是通用的：

* 应用程序(QQ文本框)获得焦点事件。
* 获得用户按了哪些键。
* 获得光标位置，显示候选词。
* 将用户选中的文字传送到应用程序。

将这些功能抽象出来就是我们所说的输入法框架。输入法框架提供了一套可以完成上述功能
的API。所以一般要做一款输入法，只需要针对目标输入法框架完成输入法引擎。
由于闭源的关系，window上的输入法框架就只有微软自己的，xp及以前就是IMM，之后是TSF。
对应的，linux上的输入法框架就比较丰富了。下面简单介绍几款linux上比较流行的输入法框架。
需要注意的一点是，输入法框架是没有标准规范的。那么针对特性输入法框架编写的输入法
程序在其他框架中一般是不能通用的。

## 常用linux输入法框架
### scim
Smart Common Input Method

scmi是一个输入法平台，在类posix的操作系统如linux和bsd中，它支持至少30种语言。
它有一个非常清晰的架构，使用了一个简单但非常强大的编程接口，该接口为我们开发自己
的输入法大大的节省了时间。

### fcitx
fcitx一个支持扩展的输入法框架。目前它支持linux和unix系统如freebsd。它有三个内置
的输入法引擎，分别是拼音输入法, 区位输入法和table-based输入法。

### ibus
intelligent input bus

ibus是在类unix操作系统中是一个为了多语言输入的输入法框架，之所以叫做"bus"原因在
于它是一个总线型的架构。ibus基于dbus ipc协议。ibus也是ubuntu默认的输入法框架。

## 好用的输入法
### rime
window下的发行版叫'小狼毫'，Mac发行版叫'鼠须管'，linux发行版叫做'中州韵'。有ibus
和fcitx版本。rime已加入到ubuntu software center，对于ubuntu 12.10及以上版本可以
通过`sudo apt-get install ibus-rime`安装。
项目地址是[rime](https://code.google.com/p/rimeime/)。

## 参考
* [https://zh.opensuse.org/index.php?title=Packaging_Input_Method_Framework_and_Engines&variant=zh](https://zh.opensuse.org/index.php?title=Packaging_Input_Method_Framework_and_Engines&variant=zh)
* [http://my.oschina.net/gschen/blog/133447](http://my.oschina.net/gschen/blog/133447)
* [https://code.google.com/p/rimeime/](https://code.google.com/p/rimeime/)
