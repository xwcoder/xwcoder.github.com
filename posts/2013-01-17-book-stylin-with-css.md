# 书评：CSS设计指南（第3版）#
![stylin with css](http://img3.douban.com/mpic/s24262786.jpg)

读[这本书](http://www.ituring.com.cn/book/1111)是想尝试[图灵社区](http://www.ituring.com.cn)「在线翻译」的创新方式。
总的来说「在线翻译」的方式有助于翻译的质量，能在成书前就得到读者的反馈，这是好事，在本书的评论里就有不少反馈。
反馈里有一部分是typo，还有一部分是对屌丝浏览器的吐槽。还有一小部分是对某个点的补充和深入，比如BFC的概念，这是书里没有提及的，
但了解这个概念确实能帮助读者理解一些现象。

但是对于参与这个过程的读者来说这个方式就没有那么有益了。相反的，由于翻译可能持续的时间较长，想做检视就是不可能的了。
而且阅读一本书用的时间过长并不是一件好事。无论怎样，对于一般读者来说，成书时能拿到一本翻译更精良的书总是好的。
相信被某些无良译者翻译的语句不通的书籍折磨过的人更有体会。顺便说下，作为读者，一直对李松峰老师的翻译功力深感佩服。

说说这本书吧。

可以作为一本入门的书。大部分情况下学习一项技术是要读多本书的。入门某项技术时，一本大而全的书籍并不是好的选择，这会消磨积极性，
烦乱的知识点可能开始就把读者弄晕了。一本短小精干的书应该是比较好的选择，快速入门，甚至上手干活。随着学习的深入，再选择相应的书，
比如《css权威指南》。当对这项技术的方方面面做到胸有沟壑，需要的就只是一本参考手册了。

一般入门书都会包括两部分的内容：「是」的部分和「做」的部分。这本书也不例外。

「是」的部分讲的是知识本身，「是什么」。此书从基本的html结构讲起，顺序讲了选择器、盒模型、背景、字体文本、表单，还有部分css3的东西。
逻辑很清晰。这些内容是日常干活时用到的css的主要也是必要内容。如计数器之类这些不常用的就丝毫没有提及。当然这本书的定位是与《css权威指南》不同的。
《css权威指南》全在讲「是什么」。只讲必要的是个明智的选择。让我不解的是没有关于表格的部分，虽然经过去table运动，但table还是有它使用场景的，
而且需要使用table的场景还挺多的。还有深度的问题，比如在讲负边距的地方，如果讲下这个公式「margin + border + padding + width = 包含块的宽度」(不是严格的)
可能会使读者更容易理解利用负外边距布局是怎么回事。在这一部分的闪光点是第4章：字体文本。
讲的清晰、明白。这闪光点更多来自译者李松峰老师的文章，[揭开leading（加铅条）的面纱](http://www.ituring.com.cn/article/18076)。

再说「做]的部分。先说缺点吧，缺少某些概念和逻辑。比如讲清理浮动时，没有介绍BFC概念和IE下的hasLyaout，
从而归纳出为什么通用的清除浮动的方法是这样的。关于布局，我觉得不如[精通CSS](http://book.douban.com/subject/4736167/)讲的清晰。
还有第7章完整的例子中，没有使用属性缩写，比如font。使用缩写可以对css进行瘦身，这是一些公司笔试/面试时的一个考察点。好的方面，
也是全书最值得推荐的一点是：从0开始的一个例子。第7章从0开始实现了一下本书站点的首页。这对初学者是宝贵的实践经验。

全书最后说了下响应设计(RWD)。涵盖的也是标准的RWD必讲的两个方面：媒体查询和流式布局。相比较，本章提供的参考资料链接更有价值。
而且对于RWD，我一直觉得光靠前端能做的是有限的、不完美的。必须要结合后端，至少要到模板层。当然怎么做需要实际工作中探索积累。

总的来说，这是一本值得推荐的入门书籍。
