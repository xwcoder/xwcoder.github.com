---
layout: post
title: 视频相关概念
published: true
tags: [video-encoding video-concept mp4 h.264]
---
当我们谈到视频文件时，经常会听到/说到很多名词，像码率、码流、mp4、avi、rmvb、h.264、webM、m3u8等等。
名词太多，容易混淆。今天来梳理一下视频相关的概念。

## 码率、码流

码率、码流是同一个概念，是数据传输时单位时间传送的数据量，一般用单位kbps即千位每秒。

视频码率就是指视频文件在单位时间内使用的数据量。简单理解就是要播放一秒的视频需要多少数据，从这个角度就不难理解
通常码率越高视频质量也越好，相应的文件体积也会越大。码率、视频质量、文件体积是正相关的。但当码率超过一定数值后，
对图像的质量影响就不大了。几乎所有的编码算法都在追求用最低的码率达到最少的失真(最好的清晰度), 围绕这个诉求衍生
出了固定码率(cbr)和可变码率(vbr)。

<pre>
//码率计算公式
码率(kbps) = 文件大小 x 1024 x 8 / 时间(秒)
</pre>

## 视频容器格式和视频编码格式

通常一个视频文件会包含很多信息，比如视频图像数据、音频数据、视频题目和封面等元数据，有时还会包括字幕数据。当然这些数据
都是要按照一定规则存储的，存储的规则就是由不同的视频容器格式决定的。不同的容器格式存储数据的规则也不一样。我们平时见
到的rmvb、mp4、avi等都是容器格式,它们规定了各自能存储哪种编码的音视频数据，以及如何存储。

无损音视频文件的体积非常庞大，不便于通过网络传播，所以需要对音视频进行编码。编码的目的就是压缩文件的体积，但是伴随着文件
体积的减小清晰度也会受损。所有编解码算法都在追求更小的文件体积和更好的清晰度。所以衡量某种编解码方案优劣的重要指标就是：
相同文件体积下是否清晰度更高, 或者相同清晰度下文件体积更小。我们经常见到的h.264、AAC、Theora等就是指音视频的编解码方案(编解码算法)。

如果我们要做一款能播放mp4视频的播放器(mp4使用h.264视频和AAC音频), 那我们需要：

1. 能够解析mp4文件，从mp4文件中提取视频数据和音频数据,类似于解压tar文件。
2. 能够解码h.264视频文件，进行视频影像的播放。
3. 能够解码AAC音频文件，进行声音的播放。

当然还有很多其他工作要做，比如显示字幕，比如音画同步等。我们要理解的是，要播放视频，不光要能够解析视频容器,从中提取出各部分的数据，
还要有相应的音视频解码器。

多媒体容器格式一般都包括三部分的数据: 文件头部分、索引部分、多媒体数据部分。

<table>
    <tr><td>文件头部分</td></tr>
    <tr><td>文件索引部分</td></tr>
    <tr><td>多媒体数据部分</td></tr>
</table>

### 文件头部分

文件头部分说明了多媒体数据的压缩标准和规范信息。同一种容器可以存储不同编码的多媒体数据，所以在播放时要知道多媒体数据的编码格式，
这部分数据存放在文件头部。

常见的多媒体数据编码标准有:

1. MPEG(Moving Picture Experts Group)系列，MPEG系列包括MPEG视频，MPEG音频和MPEG系统(音视频同步)三个部分, 提供的音视频编码方案有
MPEG-1、2、4

<table>
    <caption>视频</caption>
    <tr>
        <td width="100">MPEG-1</td>
        <td>
        较早的视频编码，质量较差，主要用于CD-ROM存储视频，VCD(video CD)的视频编码就是采用的MPEG-1
        </td>
    </tr>
    <tr>
        <td>MPEG-2</td>
        <td>
        在MPEG-1基础上开发的视频编码，质量远好于MPEG-1。MPEG-2是DVD-video唯一指定的视频编码。现在大部分HDTV(高清电视)也采用MPEG-2,
        分辨率达到了1920x1080。由于MPEG-2的普及，本来为HDTV准备的MPEG-3最终宣告放弃。
        </td>
    </tr>
    <tr>
        <td>MPEG-4</td>
        <td>
        为了适应网络传输，又开发出了MPEG-4。MPEG-4采用了一系列新技术，来满足在低带宽下传输较高质量的视频。
        </td>
    </tr>
    <tr>
        <td>MPEG-4 AVC(Advanced Video Coding)</td>
        <td>
        它和MPEG-4是两种不同的编码。主要是在极低码率下MPEG-4表现并不好，而AVC更加适合低带宽传输, 在高码率下AVC的表现也更好。
        </td>
    </tr>
</table>

<table>
    <caption>音频</caption>
    <tr>
        <td width="200">MPEG-4 Audio Layer 1/2</td>
        <td>
        也就是mp1、mp2，较早的音频编码，mp3的前身，主要用于VCD、DVD、SVCD的音频编码。
        </td>
    </tr>
    <tr>
        <td>MPEG-4 Audio Layer 3</td>
        <td>
        就是mp3, 已经成为网络音频的主流格式，能在128kbps码率下接近CD音质。
        </td>
    </tr>
    <tr>
        <td>MPEG-2 AAC</td>
        <td>
        在MPEG-2上开发的一种新的音频编码，和传统的MPEG Audio不兼容，理论上质量高于MP3, 在96kbps的码率下就能接近CD的音质，比mp3更
        适合传输。
        </td>
    </tr>
    <tr>
        <td>MPEG-4 AAC</td>
        <td>
        AAC已经作为MPEG-4标准的音频编码。
        </td>
    </tr>
    <tr>
        <td>MPEG-4 AAC Plus</td>
        <td>
        采用了SBR频带复制技术的AAC，SBR技术能够让音频编码降低一半的码率而音质不会有太大的改变，已经成为MPEG-4标准的一部分。
        </td>
    </tr>
</table>

其他还有MPEG-4 VQF、MP3 PRO、MP3 Surround。

2. h.26x系列

h.26x系列是由"ITU国际电信联盟"主导的编码系列。

<table>
    <tr>
        <td>h.261</td>
        <td>
        h.261是ITU-T为在综合业务数字网(ISDN)上开展双向声像业务(可视电话、视频会议)而制定的，它是最早的运动图像压缩标准。
        </td>
    </tr>
    <tr>
        <td>h.263</td>
        <td>ITU-T为低于64kb/s的窄带通信信道制定的视频编码标准，是在h.261基础上发展起来的。</td>
    </tr>
    <tr>
        <td>h.263+</td>
        <td>
        h.263的第二个版本。
        </td>
    </tr>
    <tr>
        <td>h.263++</td>
        <td>
        在h.263+上增加了几个选项，来增强码流在恶劣信道上的抗误码性能，同时提高编码效率。
        </td>
    </tr>
    <tr>
        <td>h.264</td>
        <td>
        就是MPEG-4 AVC。h.264是由ISO/IEC和ITU-T组成的联合视频组(JVT)制定的新一代视频压缩编码标准。在ISO/IEC中该
        标准被命名为AVC，作为MPEG-4标准的第10部分，又被成为MPEG-4 Part 10; 在ITU-T中正式命名为h.264标准。
        </td>
    </tr>
</table>

作为明星格式，h.264是目前最流行的视频编码格式。它有很多优点，编码后的文件体积小，画质高。蓝光技术(Blue-ray)就采用
这种格式。h.264的硬件解码器随处可见，几乎所有的高清摄像机、iphone、ipad、android手机上都含有h.264硬件解码器。

多媒体数据符合的规范信息可以包括视频的分辨率、帧率、音频的采样率等。

### 索引部分

多媒体数据通常会分为若干块，各块数据的储存可能是不连续的，因此需要建立索引。这样可以方便进行进度拖动等操作。视频文件
通常都比较大，播放时不能全度读入内存，拖动时可以根据索引信息加载对应的多媒体数据。

### 多媒体数据部分

多媒体数据部分就是经过压缩的多媒体数据，包括视频数据、音频数据、文本数据等。

### 常见的多媒体容器

<table>
    <tr>
        <td>MPG/MPEG</td>
        <td>
        MPEG编码采用的容器，具有流的特性。又分为PS、TS等，PS主要用于DVD存储，TS主要用于HDTV。
        </td>
    </tr>
    <tr>
        <td>AVI</td>
        <td>
        最常见的音视频容器。它可以容纳多种类型的视频编码和音频编码，想VP6、DivX、XviD等视频编码和PGM、mp3、AC3等音频编码。
        </td>
    </tr>
    <tr>
        <td>VOB</td>
        <td>DVD采用的容器格式，支持多视频多音轨多字幕章节等。</td>
    </tr>
    <tr>
        <td>mp4</td>
        <td>
        MPEG-4编码采用的容器，基于QuickTime MOV开发，具有许多先进特性。
        </td>
    </tr>
    <tr>
        <td>ASF/WMV</td>
        <td>Windows Media采用的容器，能够用于流传送，还能包含脚本等。</td>
    </tr>
    <tr>
        <td>RM/RMVB</td>
        <td>RealMedia采用的容器，用于流传送。</td>
    </tr>
    <tr>
        <td>MOV/QT</td>
        <td>QuikTime的容器</td>
    </tr>
    <tr>
        <td>mkv</td>
        <td>
        能把Windows Media Video, RealVideo, MPEG-4等视频音频融为一个文件。
        </td>
    </tr>
    <tr>
        <td>MAV</td>
        <td>一种音频容器，常说的MAV就是没有压缩的PCM编码，其实MAV还可以包括mp3等其他ACM压缩编码</td>
    </tr>
    <tr>
        <td>3GP</td>
        <td>
        3GPP视频采用的格式，主要用于流媒体传送。3GPP的视频采用了MPEG-4和h.263两种编码，音频方面音乐采用AAC，语音采用
        AMR。
        </td>
    </tr>
    <tr>
        <td>OGG</td>
        <td>
        ogg项目采用的容器，具有流的特性，支持多音轨，章节，字幕等。
        </td>
    </tr>
    <tr>
        <td>OGM</td>
        <td>Ogg容器的变种</td>
    </tr>
    <tr>
        <td>NSV</td>
        <td>Nullsoft Video的容器，用于流传送。</td>
    </tr>
</table>

## HTML5视频格式

html5提供了一个video标签，使页面可以不用依赖第三方插件(比如flash)就可以播放视频。但是规范并没有规定浏览器可以播放哪种
格式的视频。

目前h.264是最流行的视频编码格式。但是h.264是一种专利格式，它的专利被一家称为MPEG-LA的公司控制。MPEG-LA专门负责管理与
h.264有关的"专利池"。任何企业要使用h.264必须要向MPEG-LA申请许可证。MPEG-LA规定,任何放到互联网上免费播放的视频都可以无偿
获得许可证。但是这种许可证并不是永久免费的。也许未来的某一天MPEG-LA就会对其进行收费。一些主要的大公司，比如苹果和微软，
它们本身就是MPEG-LA"专利池"的所有者，所以很自然safari和ie就支持h.264视频的播放。

一些人对这种情况感到不满，于是他们开发了一种没有专利的视频格式，就是Theora。Theora的主要开发者也是Ogg Vorbis(一种
开源的、无专利的音频格式)。从Theora1.1开始，其编码效果已经不逊于h.264,尤其在低码率的情况下。但是Theora的使用并不是
很广泛，基于Theora的硬件解码器也很少。这是因为没有一家公司愿意承担Theora的专利责任，也许未来某一天就会惹上专利官司，
而且像苹果、微软已经是MPEG-LA"专利池"的所有者。像firefox这种没有资源购买h.264许可证的开源浏览器使用Theora是个不错的
选择。

Google收购了On2 Technologies后，将其拥有的VP8视频编码以类似BSD的授权开源。并基于MKV开发了一种新的容器格式WebM,
WebM采用VP8及后续VP9视频编解码器和Vorbis音频编解码器，重要的是WebM也是开源、免费的。同样属于google产品的chrome
肯定是支持WebM的。由于具有优秀的编码效果并且开源免费，相信未来WebM会得到更多浏览器的支持。

目前各浏览器对各视频格式的支持情况如下：

<table>
    <tr>
        <td>格式</td>
        <td>IE</td>
        <td>chrome</td>
        <td>firefox</td>
        <td>safari</td>
        <td>opera</td>
    </tr>
    <tr>
        <td>mp4</td>
        <td>9.0+</td>
        <td>5.0+</td>
        <td>No</td>
        <td>3.0+</td>
        <td>No</td>
    </tr>
    <tr>
        <td>WebM</td>
        <td>No</td>
        <td>6.0</td>
        <td>4.0+</td>
        <td>No</td>
        <td>10.6+</td>
    </tr>
    <tr>
        <td>Ogg</td>
        <td>No</td>
        <td>5.0+</td>
        <td>3.5+</td>
        <td>No</td>
        <td>10.5+</td>
    </tr>
</table>

<table>
    <tr>
        <td>mp4</td>
        <td>h.264视频、AAC音频</td>
    </tr>
    <tr>
        <td>WebM</td>
        <td>VP8视频、Vorbis音频</td>
    </tr>
    <tr>
        <td>Ogg</td>
        <td>Theora视频、Vorbis音频</td>
    </tr>
</table>

## m3u8

m3u8文件是m3u文件的一种，只不过它的编码格式是utf-8。m3u使用Latin-1字符集编码。
m3u的全称是Moving Picture Experts Group Audio Layer 3 Uniform Resource Locator,即mp3 URL。m3u是纯文本文件，可以理解为
简单的播放列表。m3u文件里存放的是一个一个的视频地址。打开文件编辑器，输入几首硬盘上的mp3文件路径(比如~/sing-1.mp3), 
每个文件一行，然后将文件保存为list.m3u,用关联的播放器就可以打开这个播放列表进行播放。

### m3u8文件格式解析

<pre>
#EXTM3U  标头，此句必须在文件的第一行 
#EXT-X-VERSION  该属性是可选的
#EXT-X-STREAM-INF {属性}
    BANDWIDTH       指定码率 
    PROGRAM-ID      唯一ID 
    CODECS          指定流的编码类型 
#EXT-X-TARGETDURATION   定义每个视频文件的最大的持续时间。 
#EXT-X-MEDIA-SEQUENCE   定义当前m3u8文件中第一个文件的序列号，每个视频文件在m3u8文件中都有固定
                        唯一的序列号 ，该序列号用于在MBR时切换码率进行对齐。 如果没有默认是0
#EXT-X-KEY 表示怎么对视频进行解码。
#EXT-X-PROGRAM-DATE-TIME    第一个文件的绝对时间 
#EXT-X-ALLOW-CACHE  是否允许cache。 
#EXT-X-ENDLIST 表明m3u8文件的结束。live m3u8没有该tag。 
#EXT-X-STREAM-INF {属性}
    BANDWIDTH         指定码率 
    PROGRAM-ID        唯一ID 
    CODECS            指定流的编码类型 
#EXT-X-DISCONTINUITY 当遇到该tag的时候说明以下属性发生了变化 
</pre>

m3u8文件分为两种: 顶级m3u8文件和二级m3u8文件。
顶级m3u8文件是一组二级m3u8文件的列表，用于针对不同带宽切换合适的码流。二级m3u8文件里包含的就是真正
的视频列表。

可以没有顶级m3u8文件，只使用二级m3u8文件。m3u8文件也最多嵌套两层。

<pre>
顶级m3u8文件:
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=201273221265,BANDWIDTH=358400
1.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=201273221265,BANDWIDTH=972800
2.m3u8
</pre>

客户端可以根据不同的带宽来切换播放1.m3u8和2.m3u8

<pre>
二级m3u8文件：
#EXTM3U
#EXT-X-TARGETDURATION:30
#EXT-X-VERSION:3
#EXTINF:10,
http://123.125.123.82/ipad?file=/5/233/RveYSM3BT9XVjcMMa58RQE.mp4&start=0&end=9.92&ch=tv&cateCode=101100;101107;101111&uid=1408082300056447&plat=h5&pt=2&prod=h5&pg=1&vid=1983841&eye=0&sig=loyj8ZSU2CNsbGb9ygFqvcH-Gey2B6C1
#EXTINF:26.76,
http://123.125.123.82/ipad?file=/5/233/RveYSM3BT9XVjcMMa58RQE.mp4&start=9.92&end=36.68&ch=tv&cateCode=101100;101107;101111&uid=1408082300056447&plat=h5&pt=2&prod=h5&pg=1&vid=1983841&eye=0&sig=loyj8ZSU2CNsbGb9ygFqvcH-Gey2B6C1
#EXTINF:29.84,
http://123.125.123.82/ipad?file=/5/233/RveYSM3BT9XVjcMMa58RQE.mp4&start=36.68&end=66.52&ch=tv&cateCode=101100;101107;101111&uid=1408082300056447&plat=h5&pt=2&prod=h5&pg=1&vid=1983841&eye=0&sig=loyj8ZSU2CNsbGb9ygFqvcH-Gey2B6C1
#EXT-X-ENDLIST
</pre>

#EXTINF: {数值}, 标示当前视频段的时常，这样便于进行进度拖放和切换清晰度后播放进度的定位。

当前国内的主流视频网站都只使用二级m3u8文件，将切换清晰度的功能提供给用户。m3u8文件非常适合cdn的分发。

## 参考资料

[HTML5的视频格式之争](http://www.ruanyifeng.com/blog/2010/05/html5_codec_fight.html)   
[视频码率](http://baike.baidu.com/view/1319178.htm)   
[多媒体容器与压缩标准的概念区别](http://www.360doc.com/content/10/1225/14/1053846_81203652.shtml)   
