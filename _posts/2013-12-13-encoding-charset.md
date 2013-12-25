---
layout: post
title: 前端工程师的编码问题
published: true
tags: [encoding charset unicode ucs UTF-8 UTF-16 编码 字符集]
---

这篇文章分两个部分。第一部分梳理一下字符集和编码的基础知识，比如unicode、ucs、utf-8、utf-16之间的关系，
在这部分中主要以文本文件为例，相关内容同样适用于字节流,
第二部分整理一下前端工程师可能会遇到的一些编码问题。我的愿望是想能穷尽所有问题，但是我又知道这是不可能的，
所以只能尽量多的把搜集到的以及遇到过的问题整理出来。

## 基本概念

### 字符集与编码

通常我们讲到的字符是一种文化符号，比如'中华'二字，比如甲骨文。顾名思义，字符集就是一套字符的集合。

计算机只能处理/存储二进制数据，在计算机内部所有数据都是由0,1组成的字节串。那么怎么用计算机来显示和处理人类文明使用的字符呢？
这就需要对字符进行编码，简单来说就是建立二进制数字和字符之间的对应关系。比如在
ASCII这套字符集中'01000001'(65)就代表大写字母'A'。

在计算机的范畴内, 字符集编码通常包含了两部分的内容：1、一套字符集
2、这套字符集与二进制数字的对应关系，这套对应关系叫做码表。
比如繁体字'華'就不会在GB2312的码表中，因为设计GB2312时就没有把繁体'華'字收录进去。

前面说到了字符集和码表的概念。现在说说字符怎么存储与读取。
为方便说明，后文主要以文本文件存储到硬盘和从硬盘中读取文件并显示为例。出于节省存储空间等方面的考虑，
通常字符并不是按照码表中对应的二进制直接存储的，而是将这个二进制串再经过一次运算得到另一个更短的二进制串，
最终存储的是这个更短的二进制串。这有点类似twitter和微博等使用的短域名。当然也可能不经过转换。

文本文件的大致存储过程是这样的：

1. 查找码表，得到字符对应的二进制串A
2. 将A按一定规则进行转换得到二进制串B
3. 将二进制B存储到硬盘

注意：步骤2可能没有。

读取文本文件就是相反的过程：

1. 从硬盘中读取数据，得到二进制串B
2. 将B按一定规则进行转换得到二进制串A
3. 查找码表，找到二进制串A对应的字符，并显示字符

注意：步骤2可能没有。

这个世界上存在着很多不同的字符集和各自相应的编码方式，比如ASCII，比如GB2312,GBK,
GB18030，同一套字符集可能会有不同的编码方式，比如UTF-8,UTF-16都使用unicode这套字符集。

当然你也可以设计自己的字符集，比如设计一套字符集命名为前端字符集，
这套字符集只包括“前端工程师”这五个简体中文字，分别使用'0000000','0000001','0000010',
'0000011','0000100'表示，当读取数据时每次取一个字节，然后在码表中找到对应的字符，
如果读取的字节不是这5个字节的任意一个，那么就显示为乱码。

好了，介绍了字符、字符集、码表的基本概念，下面来看看中文程序员用到最多的几种编码。
(据我观察在程序员的日常交流中，很多时候字符集、字符集编码、编码泛指一个概念)

### ASCII

最初，计算机只在美国使用，由于英文符号比较少，美国人就使用1个字节来对他们使用到的字符进行编码，
这就是ASCII码(American Standard Code for Information Interchange，美国信息交换标准代码)。

1个字节可以表示256种不同的状态(2^8=256)，也就是说ASCII最多包含256个字符。
其实这256个状态都没用完，只使用到了第127号(0x7F)，其中从0开始的32种状态被用做一些特殊用途。
比如终端遇到0x07就会发出叫声。这些0x20以下的状态被成为'控制码'。

### ISO-8859-1

> ISO 8859-1，正式编号为ISO/IEC 8859-1:1998，又称Latin-1或“西欧语言”，
> 是国际标准化组织内ISO/IEC 8859的第一个8位字符集。它以ASCII为基础，在空置的0xA0-0xFF的范围内，
> 加入96个字母及符号，藉以供使用附加符号的拉丁字母语言使用。 [[wikipedia]](http://zh.wikipedia.org/wiki/ISO/IEC_8859-1)

简单来说，ISO-8859-1就是在ASCII的基础上又加了些字符，使用了一些ASCII没有使用的状态，
它是ASCII的一种扩展字符集。其实后面要讲到的GB2312,GBK,GB18030都是对ASCII的扩展。

### GB2312和GBK和GB18030

等到中国人使用计算机时发现ASCII里没有中文字符，而且一个字节最多编码256个字符，
远远不能满足中文的需要，中文的常用汉字就有6000多个。于是聪明的国人干脆就用两个字节来表示汉字。
规定：   

1. 一个不大于127的字节与ASCII中对应的字符相同
2. 两个连续大于127的字节一起表示一个中文字符(汉字或符号)
3. 前面的一个字节（高字节）从0xA1用到0xF7，后面一个字节（低字节）从0xA1到0xFE。

这套字符集和编码方案就是GB2312。

当读取数据时，先取一个字节A，如果A不大于127，A就与其在ASCII中表示相同的含义(控制码或者字符);
如果A大于127，那么再取下一个字节B，如果B也大于127，那么AB一起表示一个汉字，在码表中查询其对应的字符。
如果码表中没有AB对应的字节，或者B小于127，那么就显示乱码或者进行出错处理。

后来GB2312还是不够用，那么在编码时就放开了对低字节的要求(不一定要大于127)，
只要高字节大于127，连在一起的两个字节就表示一个'中文字符'。这样就多出了很多空间，
就可以收录更多的字符，这就是GBK。

> 由于GB 2312-80只收录6763个汉字，有不少汉字，如部分在GB 2312-80推出以后才简化的汉字（如“啰”），
> 部分人名用字（如中国前总理朱镕基的“镕”字），台湾及香港使用的繁体字，日语及朝鲜语汉字等，并未有收录在内。
> 于是厂商微软利用GB 2312-80未使用的编码空间，收录GB 13000.1-93全部字符制定了GBK编码。[[wikipedia]](http://zh.wikipedia.org/wiki/%E6%B1%89%E5%AD%97%E5%86%85%E7%A0%81%E6%89%A9%E5%B1%95%E8%A7%84%E8%8C%83)

**tips：**微软的CP936通常被视为等同GBK，事实上比较起来， GBK比CP936多收录95个字符（15个非汉字及80个汉字）。

GB2312，GBK使用两个字节表示一个汉字，通称做"DBCS"(Double Byte Charecter Set 双字节字符集)。
这就是刚学程序设计时老师总要我们记住的'一个中文字符等于两个英文字符'。

再后来，GBK还是不够用，因为中国还有很多少数民族，不少民族都有自己的文字。
于是人们又扩展出了GB18030。

> GB 18030，全称：国家标准GB 18030-2005《信息技术　中文编码字符集》，是中华人民共和国现时最新的内码字集，
> 是GB 18030-2000《信息技术　信息交换用汉字编码字符集　基本集的扩充》的修订版。与GB 2312-1980完全兼容，
> 与GBK基本兼容，支持GB 13000及Unicode的全部统一汉字，共收录汉字70244个。[[wikipedia]](http://zh.wikipedia.org/wiki/GB_18030)

GB18030收录了国内少数民族的文字，汉字部分收录了繁体汉字和日韩汉字，编码是变长的，
每个字符由1个、2个或者4个字节组成。

### unicode和ISO-10646

世界上存在如此多的字符集编码，给交流造成了极大的困难。于是有人意识到有必要给全人类设计一套统一的字符集了。

不幸的是有两个独立的组织分别开始了这项工作，一个是国际标准化组织(ISO)，项目是ISO-10646;
另一个是由Apple，Xerox等公司于1988年组成的统一码盟(unicode.org)，项目就是unicode。

幸运的是，不久他们就发现了彼此的存在，两个项目的参与者都认识到，世界不需要两个不兼容的字符集。
于是它们开始合并双方的工作成果，并为创立一个单一码表而协同工作。
1991年，不包含CJK统一汉字集的Unicode 1.0发布。随后，CJK统一汉字集的制定于1993年完成，
发布了ISO 10646-1:1993，即Unicode 1.1。

> 两个项目仍都独立存在，并独立地公布各自的标准。但统一码盟和ISO都同意保持两者标准的码表兼容，
> 并紧密地共同调整任何未来的扩展。
> [[wikipedia]](http://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E5%AD%97%E7%AC%A6%E9%9B%86)

也就是说，世界上存在两个统一字符集标准，但这两个标准是高度同步的，码表是一致的。
日常交流和技术书籍中使用unicode这个名字多一些。

unicode和ISO-10646定义的字符集就叫做通用字符集(Universal Character Set，UCS)。

unicode 1.x版本使用2个字节(16个位)编码，这就是**UCS-2**。
从1996年发布unicode 2.0开始使用4个字节编码，这就是**UCS-4**。

UCS-2最多能表示2^16=65536个字符。UCS-4中只使用了31位，最高位为0。UCS-4在已使用的31位中，
按照最高字节可以分为2^7=128个group，每个group根据次高字节分为256个panel，
每个panel根据第三高字节分为256个row，每行有256个cells。
第0号group的第0号panel被称作**BMP**(Basic Multilingual Plane)。
很明显BMP去掉高位的两个零字节就得到UCS-2。

在码表上每个字符对应的值称作code point。也可以叫做裸码，意思是没有经过转换的码值。

### UTF-8、UTF-16、UTF-32和BOM

在介绍这部分之前，先简单介绍一下IETF。IETF(Internet Engineering Task Force)中文名是互联网工程任务组，
负责互联网标准的开发和推动。当前绝大多数国际互联网技术标准出自IETF，
比如著名的http协议，代号是RFC2616。比如前端工程师都熟悉的JSON格式也纳入到了IETF的维护, 
代号是RFC4627。

#### UTF-8

UTF的全称是'Unicode Transformation Format'。

UTF-8的全称是'8-bit  Unicode Transformation Format'。由IETF维护，代号是[RFC3629](http://www.ietf.org/rfc/rfc3629.txt)。
UTF-8是unicode(ISO-10646)的一种存储/传输方式。还记得前文中说到的文本文件存储的大致过程吗，
简单理解UTF-8就是第二步中的转换算法，由二进制串A(unicode code point)经过计算(UTF-8)得到二进制串B。

UTF-8的编码规则很简单，参照如下对照表。(表一)
<pre>
Char. number range     |        UTF-8 octet sequence
      (hexadecimal)    |              (binary)
   --------------------+-----------------------------------------
   0000 0000-0000 007F | 0xxxxxxx
   0000 0080-0000 07FF | 110xxxxx 10xxxxxx
   0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
   0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
</pre>

以简体中文'中'字为例，按照RFC3629的描述，编码过程是：

1. 通过查表得到'中'的code point是4e2d，二进制形式是'100111000101101'
2. 对照表一第一列，'中'在第三行，对应的使用的UTF-8形式是1110xxxx 10xxxxxx 10xxxxxx
3. 从低位到高位，使用'100111000101101'填充1110xxxx 10xxxxxx 10xxxxxx中的x，
未被填充的用0填充，最后得到'111001001011100010101100'

一个简单的规律就是：

1. 对于单字节的符号(高位字节全部为0)，字节的第一位设为0，后面7位为这个符号的unicode码值。因此对于英语字母，UTF-8编码和ASCII码是相同的。
2. 对于n字节的符号（n>1），第一个字节的前n位都设为1，第n+1位设为0，后面字节的前两位一律设为10。
剩下的未使用的二进制位，从低位到高位填充字符的unicode码。未填充的用0补齐。

解码就是相反的过程：

1. 读取第一个字节'11100100'，开头3个1说明一共是三个字节，然后再读取两个字节'10111000', '10101100'
2. 按照规则提取第一个字节除去开头'1110'的部分，得到'0100';第二、三个字节除去开头'10'的部分，得到'111000', '101100'
3. 将这个三部分拼接到一块得到'0100111000101100'，查表得到是'中'。

编码也会遇到撞码的情况：当我们以一种编码方式A存储数据得到二进制格式A1，而以另一种编码方式B读取数据A1，
恰好A1也符合B的编码方式。还记得著名的'联通'烧焦电池的问题吧：在window下新建文本文件，然后输入'联通'二字保存，
再用记事本程序打开，'联通'二字不见了，取而代之的是一个烧焦电池的图形。这其实就是撞码造成的。
让我们来分析一下这个问题，

* 首先，新建文本文件时默认的格式是ANSI，对于英文文件是ASCII编码，
对于简体中文文件是GB系列编码(只针对Windows简体中文版，如果是繁体中文版会采用Big5码)
* 输入'联通'后保存文件是GB系列编码的，以十六进制查看是<code>c1 aa cd a8</code>，对应的二进制是

        parseInt( 'c1aacda8', 16 ).toString( 2 ) // 11000001 10101010 11001101 10101000
* 当再次打开文件时，记事本程序首先会尝试按UTF-8解析数据，恰巧这4个字节符合UTF-8的编码规则，
前两个字节转换得到1101010，对应的字符是'j'，后两个字节转换得到1101101000，
unicode没有这个code point，所以显示为乱码。
* 当打开文件时，文本编辑器都会尝试用不同的编码格式读取数据，当首选的编码格式读取有错误就会尝试使用其他的格式，
比如vim都会设置

        set fencs=utf-8,gb18030,gbk,gb2312,cp936,ucs-bom,euc-jp,
当尝试使用utf-8失败时就会使用gb18030，以此类推。如果将utf-8设置在gb2312之后，就能"正常打开"。
同样这也解释了:多保存'移动'两个字后就不会出现乱码，因为移动按GB系编码后不会与utf-8撞码，
当试图使用utf-8读取时会发生错误，文本编辑器就会尝试使用其他的编码方式读取。

#### UTF-16

UTF-16的RFC编号是2781，[RFC2781](http://www.ietf.org/rfc/rfc2781.txt)

假设U是某字符的unicode code point，UTF-16的编码原则是：

1. 如果U < 0x10000, 就使用两个字节表示U的值
2. 如果0x10000 <= U <= 0x10FFFF，就使用两个16-bit表示(4字节),
前两个字节取值范围在0xD800到0xDBFF, 被称作high-half zone。
后两个字节取值范围在0xDC00到0xDFFF, 被称作low-half zone。
3. 如果U > 0x10FFFF, 那么不能使用UTF-16进行编码。

详细的编码流程是：
1. 如果U < 0x10000，使用两个字节表示U的值。结束。
2. 令U' = U - 0x10000. 因为U <= 0x10FFFF, 那么U' <= 0xFFFFF, 所以U'就可以用20个bit表示。如需要，高位用0不齐。
3. 使用两个16-bit，w1和w2, 令w1=0xD800, w2=0xDC00，每一个16-bit中有10个bit可以用于编码，一共有20个bit位可供使用编码。
4. 将U'(20-bit表示的)高位的10个bit填充到w1中，将U'低位的10个bit填充到w2中。结束。

图形表示这个过程如下:
<pre>
U' = yyyyyyyyyyxxxxxxxxxx
W1 = 110110yyyyyyyyyy
W2 = 110111xxxxxxxxxx
</pre>

解码流程是:
读取两个字节(16-bits)w1，读取紧随w1的两个字节w2(16-bits)。

1. 如果w1 < 0xD800或者w1 > 0xDFFF, w1的值就是code point。结束。
2. 判断w1是否在0xD800和0xDBFF之间, 如果不在，解析报错。结束。
3. 如果没有w2或者w2不在0xDC00和0xDFFF之间，解析报错。结束。
4. 用w1的后10个bit和w2的后10个bit组成U'(20个bit)。
5. 令U = 0x10000 + U'，得到U。结束

#### UTF-32

UTF-32就是使用4个字节表示code point，无需像UTF-16和UTF-8那样进行转换，就是unicode的裸码。

#### BOM

不同CPU处理多字节的方式是不同的，分为Big endian(大头方式)和Little endian(小头方式)。
Big endian认为第一个字节是最高位字节，Little endian则相反。比如'奎'的unicode码是594E，
'乙'的unicode码是4E59，当一个处理UTF-16文本的程序读到字节流'594E'时，
那么怎么区分是'奎'还是'乙'呢？这就需要BOM了。

我们可以在字节流的开头加入两个特殊字节来表示字节的顺序，这两个特殊字节就是FF FE，
如果字节流的头两个字节是FEFF就表示是Big endian，如果头两个字节是FFFE就表示是Little endian。
这两个字节就是BOM(byte-order mark)—字节顺序标记。
在一些旧的文档中会说到U+FEFF这个字符是'Zero Width No-Break Space'(零宽度非换行空格)，
现在零宽度非换行空格有了新的定义，来看一下wikipedia上的说明。

> 字符U+FEFF如果出现在字节流的开头，则用来标识该字节流的字节序，是高位在前还是低位在前。
> 如果它出现在字节流的中间，则表达零宽度非换行空格的意义，用户看起来就是一个空格。
> 从Unicode3.2开始，U+FEFF只能出现在字节流的开头，只能用于标识字节顺序，
> 就如它的名称——字节顺序标记(BOM)——所表示的一样；除此以外的用法已被舍弃。
> 取而代之的是，使用U+2060来表达零宽度非换行空格。[[wikipedia]](http://zh.wikipedia.org/wiki/%E4%BD%8D%E5%85%83%E7%B5%84%E9%A0%86%E5%BA%8F%E8%A8%98%E8%99%9F)

所以UTF-16,UTF-32分别还有两个格式UTF-16BE，UTF-LE，UTF-32BE，UTF-32LE。
一些技术书籍和文档中说UTF-16是长度不变的编码方式，这种说法是不正确的。

表二，
<pre>
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Name                       | UTF-8  | UTF-16  | UTF-16BE   | UTF-16LE      | UTF-32  | UTF-32BE   | UTF-32LE      |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Smallest code point        | 0000   | 0000    | 0000       | 0000          | 0000    | 0000       | 0000          |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Largest code point         | 10FFFF | 10FFFF  | 10FFFF     | 10FFFF        | 10FFFF  | 10FFFF     | 10FFFF        |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Code unit size             | 8 bits | 16 bits | 16 bits    | 16 bits       | 32 bits | 32 bits    | 32 bits       |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Byte order                 | N/A    | [BOM]   | big-endian | little-endian | [BOM]   | big-endian | little-endian |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Fewest bytes per character | 1      | 2       | 2          | 2             | 4       | 4          | 4             |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
| Most bytes per character   | 4      | 4       | 44         | 4             | 4       | 4          | 4             |
|----------------------------|--------|---------|------------|---------------|---------|------------|---------------|
</pre>

UTF-8编码的文件是没有字节顺序问题的，那么可以用BOM来标识此文件是否为UTF-8编码的。
当文件的前三个字节是EF BB BF，就表示此文件是UTF-8编码的。
很多windows程序(比如记事本)采用这种方式。但是在类Unix系统中，这种作法则不被建议采用。
因为对无法识别这三个字符的程序会产生影响。

好了，到现在讲了字符集、码表的相关知识，也粗略介绍了几种常用的字符集和编码方式。
接下来看看前端工程师经常使用到的编码知识和经常遇到的编码问题。

## 前端工程师经常遭遇的编码问题

前端工程师会遇到很多编码问题，比如静态文件的编码，数据接口的编码，提交数据的编码等等。

### javascript的内码

像java一样，javascript使用的内码是unicode。更准确一点说是使用的UCS-2
(没找到直接规范说明，通过实验得到，可能不准确)，
javascript中，一个unicode code point前边加上'\u'就标示相应的字符。所以'\u4e2d'与'中'是相等的。

    '\u4e2d' == '中' //true

通常js压缩工具也会把程序中的中文翻译成unicode码的形式。

如果你愿意，甚至可以这样写代码：

    var \u0069 = 5 // var i = 5;
    alert( i ) // 5
    
一个将字符串转换成相应unicode格式串的函数：

    function toUnicodeHex ( s ) { 
        if ( !s ) {
            return '';
        }
        var r = [];
        for ( var i = 0, len = s.length; i < len; i++ ) {
            r[ i ] = ( '00' + s.charCodeAt( i ).toString( 16 ) ).slice( -4 );
        }
        return '\\u' + r.join( '\\u' );
    }

### escape和encodeURI和encodeURIComponent

js脚本对字符串进行编码主要用到三个函数：escape, encodeURI, encodeURIComponent。

#### escape

escape的编码规则如下：

1. 除ASCII字母、数字、标点符号'@ * _ + - . /'以外，对其他所有字符进行编码
2. 对于编码字符，使用unicode(UCS-2)码表查找裸码，并且舍弃全部为0的高字节, 得到U。
3. 对于U+0000和U+00FF之间的字符，对U加前缀'%'，其他字符加前缀'%u'。
(对于高位为0的字符, U加前缀'%'；高位非0的字符，U加前缀'%u')

        escape( '中' ) // "%u4E2D" '中'的裸码是U+4e2d
        escape( 'ü' ) // "%FC"  'ü'的裸码是U+00fc

相应的解码函数是unescape。

#### encodeURI

encodeURI是js真正用来对URL进行编码的函数。除了常见符号，
对一些在网址中有特殊含义的字符也不进行编码，比如'; / ? : @ & = + $ , #'。

encodeURI采用UTF-8的方式进行编码，然后在编码后的每个字节前加上'%'。

        encodeURI( 'http://so.tv.sohu.com/mts?wd=生活爆炸' );
            "http://so.tv.sohu.com/mts?wd=%E7%94%9F%E6%B4%BB%E7%88%86%E7%82%B8"

相应的解码函数是decodeURI。

#### encodeURIComponent

encodeURIComponent与encodeURI的区别是，它对URL的组成部分进行编码，
所以encodeURI不会编码的一些字符，encodeURIComponent会行进编码，
比如'; / ? : @ & = + $ , #'，编码方式与encodeURI是一致的。

相应的解码函数是decodeURIComponent。

三个函数不进行编码的字符见下表。
<pre>
|--------------------|-----------------------------------------------------|
|                    | none encode characters                              |
|--------------------|-----------------------------------------------------|
| escape             | * + - . / @ _ 0-9 a-z A-Z                           |
|--------------------|-----------------------------------------------------|
| encodeURI          | ! # $ & ' ( ) * + , - . / : ; = ? @ _ ~ 0-9 a-z A-Z |
|--------------------|-----------------------------------------------------|
| encodeURIComponent | ! '( ) * - . _ ~ 0-9 a-z A-Z                        |
|--------------------|-----------------------------------------------------|
</pre>

### url中含有中文

在[RFC1738](http://www.ietf.org/rfc/rfc1738.txt)中对url编码做了如下规定：

> "...Only alphanumerics [0-9a-zA-Z], the special characters "$-_.+!*'()," 
> [not including the quotes - ed], and reserved characters used for their 
> reserved purposes may be used unencoded within a URL." 
> “只有字母和数字[0-9a-zA-Z]、一些特殊符号“$-_.+!*'(),”
> [不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。”

这只规定了出现在url中的哪些字符不用编码，但是没有规定对于需要编码的字符应该使用哪种编码方式，
而是交给应用程序(浏览器)自己处理。至于各主要浏览器怎么处理，阮一峰在[关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)
这篇文章中做了总结，本文这部分内容主要来自阮一峰的这篇文章，只验证了IE和firefox。

#### 情况1：网址路径中包含汉字

例如<code>http://zh.wikipedia.org/wiki/春节</code>    
**结论是：**网址路径的编码，用的是utf-8编码，每个字节前加'%'
    http://zh.wikipedia.org/wiki/%E6%98%A5%E8%8A%82

#### 情况2：查询字符串包含汉字

例如<code>http://www.baidu.com/s?wd=春节</code>    
**结论是：**查询字符串的编码，用的是操作系统的默认编码，比如GB系。
不同点是firefox会在每个字节前加'%'。

#### 情况3：GET方法生成的URL包含汉字

例如<code><a>http://www.baidu.com/s?wd=春节</a></code>

    http://www.baidu.com/s?wd=%B4%BA%BD%DA //页面是GBK编码
    http://www.baidu.com/s?wd=%E6%98%A5%E8%8A%82 //页面是UTF-8编码
**结论是：**GET和POST方法的编码，用的是网页的编码，每个字节前加'%'。

#### 情况4：ajax调用的URL包含汉字

举例：

    url = url + "?q=" +document.myform.elements[0].value; // 假定提交的值是“春节”这两个字
    http_request.open('GET', url, true);       

**结论是：**IE总是采用GB2312编码（操作系统的默认编码），而Firefox总是采用utf-8编码。

**tip:**对于这种没有相应规范或者规范不明确的情况，怎么处理完全取决于浏览器厂商, 
所以没有必要深究。我们要做的是规避这种情况。

#### form表单

之所以form表单单独拿出来说，是因为form表单有个属性可以指定编码'accept-charset', 
比如页面是gbk编码，指定<code>accept-charset="utf-8"</code>, 提交时数据会被按照utf-8进行编码。

低版本的IE是不支持accept-charset属性的，一个替代方案是提交前将document.charset设置为需要的编码，
定时一段时间再将document.charset设置为初始编码(否则使用浏览器前进后退按钮，或者页面内锚点跳转时，
IE会使用新的编码重新渲染页面，造成页面显示乱码)。
    
    form.onsubmit = function () {
        if ( isIE ) {
            document.charset = 'utf-8';  
            setTimeout( function () {
                document.charset = 'gbk';  
            }, 2e3 );
        }
    };

### 文件加载

当加载html、css、javasript文件时，浏览器需要知道加载文件的编码格式，然后采用相应的编码进行解析渲染/处理。
如果使用了一个'错误'的编码处理文件就会产生乱码问题，这可能是我们遇到最多的乱码情况。

#### css

浏览器应该使用哪种编码处理css文件，在css2.1规范的[第四节](http://www.w3.org/TR/CSS2/syndata.html#charset)有明确定义，
按照优先级从高到底：

1. http响应头中, content-type域的charset属性。<code>Content-Type: text/html; charset=gbk</code>
2. BOM (and/or) @charset
3. <code>&lt;link&gt;</code>标签的charset属性，<code>&lt;link charset=""&gt;</code>
4. 当前页面编码
5. 使用UTF-8处理

#### html

浏览器使用如下方式来决定html文档的编码，按优先级从高到低：

1. http响应头中, content-type域的charset属性。<code>Content-Type: text/html; charset=gbk</code>
2. html页面中的meta标签中指定charset。<code>&lt;meta charset="gbk" /&gt;</code>

关于如何从meta中提取charset，[这里](http://www.w3.org/TR/html5/infrastructure.html#extracting-character-encodings-from-meta-elements)有详细的说明。

还有一点需要说明，对于使用UTF-16编码的html文档，规范中规定应该使用BE(big-endian)方式，
为了更好的兼容性，并且推荐总是带有BOM。

对于没有进行编码设置的文档，浏览器会不会像vim一样依次尝试使用不同编码，
或者使用操作系统默认编码，或者像处理css的第5条干脆默认使用某种编码，完全取决于各浏览器厂商。
所以要避免html文档不进行编码设置的情况。

#### javascript

浏览器使用如下方式来决定js文件的编码，按优先级从高到低：

1. http响应头中, content-type域的charset属性。<code>Content-Type: text/html; charset=gbk</code>
2. &lt;script&gt;标签的charset属性。<code>&lt;script src="" charset="gbk"&gt;&lt;script&gt;</code>
3. 当前页面编码

## 结束语

欢迎补充、指正。

## 参考资料

[http://www.ruanyifeng.com/blog/2010/02/url_encoding.html](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)    
[http://zh.wikipedia.org/wiki/位元組順序記號](http://zh.wikipedia.org/wiki/%E4%BD%8D%E5%85%83%E7%B5%84%E9%A0%86%E5%BA%8F%E8%A8%98%E8%99%9F)   
[http://www.unicode.org/](http://www.unicode.org/)    
[http://www.unicode.org/faq/utf_bom.html](http://www.unicode.org/faq/utf_bom.html)    
[http://zh.wikipedia.org/wiki/Unicode](http://zh.wikipedia.org/wiki/Unicode)    
[http://zh.wikipedia.org/wiki/通用字符集](http://zh.wikipedia.org/wiki/%E9%80%9A%E7%94%A8%E5%AD%97%E7%AC%A6%E9%9B%86)    
[http://zh.wikipedia.org/wiki/ISO/IEC_8859-1](http://zh.wikipedia.org/wiki/ISO/IEC_8859-1)    
[http://zh.wikipedia.org/wiki/GB_2312](http://zh.wikipedia.org/wiki/GB_2312)    
[http://zh.wikipedia.org/wiki/汉字内码扩展规范](http://zh.wikipedia.org/wiki/%E6%B1%89%E5%AD%97%E5%86%85%E7%A0%81%E6%89%A9%E5%B1%95%E8%A7%84%E8%8C%83)    
[http://zh.wikipedia.org/wiki/GB_18030](http://zh.wikipedia.org/wiki/GB_18030)
[http://www.ietf.org/rfc/rfc3629.txt](http://www.ietf.org/rfc/rfc3629.txt)    
[http://www.ietf.org/rfc/rfc2781.txt](http://www.ietf.org/rfc/rfc2781.txt)    
[http://www.ietf.org/rfc/rfc1738.txt](http://www.ietf.org/rfc/rfc1738.txt)    
[http://www.w3.org/TR/1999/REC-html401-19991224/charset.html#h-5.2](http://www.w3.org/TR/1999/REC-html401-19991224/charset.html#h-5.2)
