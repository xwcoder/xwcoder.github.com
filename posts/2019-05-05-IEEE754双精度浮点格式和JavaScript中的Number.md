[TOC]

这篇内容尝试解释清楚IEEE 754双精度浮点格式和JavaScript中的Number类型，并简单介绍Smi。

## Number类型的定义

首先看一下[ECMA-262](https://tc39.github.io/ecma262/#sec-ecmascript-language-types-number-type)对Number类型的定义：
>The Number type has exactly 18437736874454810627 (that is, 264 - 253 + 3) values, representing the double-precision 64-bit binary format IEEE 754-2008 values as specified in the IEEE Standard for Floating-Point Arithmetic, except that the 9007199254740990 (that is, 253 - 2) distinct “Not-a-Number” values of the IEEE Standard are represented in ECMAScript as a single special NaN value.

在`BigInt`被引入之前，JavaScript中只有Number一种数值类型，采用IEEE 754双精度浮点格式表示，不区分整型和浮点型。

目前(2019-05)，`BigInt`处于[Stage 3](https://github.com/tc39/proposals)阶段，当前可以在v8中使用BigInt，Node.js中也引入了使用BigInt的API，比如`process.hrtime.bigint()`。

## 十进制到二进制转换
文中内容会涉及到相关计算，所以先再熟悉下十进制到二进制的转换计算。

> 整数部分，除2取余，直至商数为0，从下到上读余数，即是二进制的整数部分。小数部分，用其乘2，取其整数部分的结果，再用计算后的小数部分依此重复计算，算到小数部分全为0为止，从上到下读所有计算后整数部分的数字，即是二进制的小数部分。-- [wikipedia](https://zh.wikipedia.org/wiki/%E4%BA%8C%E8%BF%9B%E5%88%B6#%E5%8D%81%E8%BF%9B%E6%95%B8%E8%BD%AC%E6%88%90%E4%BA%8C%E8%BF%9B%E6%95%B8)

举例，将$59.25_{(10)}$转换为二进制：

```sh
// 整数部分：
59 ÷ 2 = 29 ... 1
29 ÷ 2 = 14 ... 1
14 ÷ 2 =  7 ... 0
 7 ÷ 2 =  3 ... 1
 3 ÷ 2 =  1 ... 1
 1 ÷ 2 =  0 ... 1
// 小数部分：
0.25×2=0.5
0.50×2=1.0
```

$59.25_{(10)}$ = $111011.01_{(2)}$
![](http://latex.codecogs.com/gif.latex?%2459.25_%7B%2810%29%7D%24%20%3D%20%24111011.01_%7B%282%29%7D%24)

不难发现，**对于小数部分最后一位是1，2，3，4，6，7，8，9的十进制数是不能转换成有限位数的二进制的**。

## 科学计数法
对于任何一个十进制数都可以用科学计数法表示，比如322000 = 3.22 x $10^5$。同理，二进制数也可以用科学计数法表示，比如$59.25_{(10)}$ = +$111011.01_{(2)}$ = +$11.101101_{(2)}$ x $2^4$ = +$1.1101101_{(2)}$ x $2^5$。
![](http://latex.codecogs.com/gif.latex?%2459.25_%7B%2810%29%7D%24%20%3D%20&plus;%24111011.01_%7B%282%29%7D%24%20%3D%20&plus;%2411.101101_%7B%282%29%7D%24%20*%20%242%5E4%24%20%3D%20&plus;%241.1101101_%7B%282%29%7D%24%20*%20%242%5E5%24)

![93496D18-A8E2-44C7-BE2D-65BF9412316D.png](https://xwcoder.github.io/resources/BA6DBDF6E90507E4DC1EF055F2D3338B.png)

**整数部分总是可以精确到1**，那么只需要记录**符号位、小数、指数位**这三个部分的值就可以完整表示一个二进制数。

## IEEE 754双精度浮点格式

### IEEE 754

IEEE标准协会（英文Institute of Electrical and Electronics Engineers Standards Association，简称IEEE-SA）是电气和电子工程师协会（IEEE）下辖的标准制定机构，其标准制定内容涵盖信息技术、通信、电力和能源等多个领域，已制定了900多个现行工业标准。

其中IEEE 754是二进制浮点数算术标准。标准中定义了二进制浮点数的格式，其中包括双精度浮点格式。
![11C4FED2-B837-4C38-83BD-59FE6554EEDB.png](https://xwcoder.github.io/resources/EE239D826207BB79794666FCE1D5D932.png)

单精度浮点格式使用32位(4字节)表示，也被称为`binary32`。双精度浮点格式使用64位(8字节)表示，也被称为`binary64`。

### 双精度浮点格式

IEEE 754二进制浮点格式有**规约**和**非规约**两种形式。这部分内容主要介绍双精度浮点格式，先介绍其规约形式，之后介绍非规约形式，最后介绍**特殊值**。

其他精度的浮点格式表示方法类似，只是在**各部分比特位位数**、**指数偏移量**等方面有差异。

#### 规约形式

![90737FF2-1CED-42D3-9D54-32C3F6F53ECC.png](https://xwcoder.github.io/resources/396F19A63CA24F45611D161C9400A44D.png)

在**科学计数法**部分介绍过，完整表示一个二进制数只需要记录**符号、指数位、小数**三部分信息。IEEE 754浮点格式就是按照**科学计数法**的方式存储值的。不同精度格式的指数和小数部分有不同的位数。

![6DF39F67-0467-4144-8690-5DE912EF2068.png](https://xwcoder.github.io/resources/48240F0E1E0DD33EC89100CBE2D30707.png)

具体到双精度浮点格式，各部分的位数如下：

* 符号位(sign bit): 1位。
* 指数位(exponent bit): 11位。
* 小数部分(fraction bit): 52位。

##### 符号位
符号位很好理解，1代表负，0代表正。

##### 指数位
指数位有11位。指数部分的值e使用如下运算规则得到：
1. 将指数部分按11位无符号整数解析得到e1，所以e1的取值范围是[0, 2047]。
2. 其中0`(00000000000)`和2047`(11111111111)`有特殊含义，另作他用；所以e1的取值范围是[1, 2046]，即编码范围`[00000000001, 11111111110]`。`00000000000`和`11111111111`会在**非规约形式**和**特殊值**中用到，后面会有介绍。
3. `e = e1 - 1023`，所以e的取值范围是[-1022, 1023]。减去的1023被称为**指数偏移量**，不同精度的指数偏移量不同，双精度浮点格式指数偏移量是1023，单精度浮点格式指数偏移量是127。

关于规则3中`e1 - 1023`以及不同精度的指数偏移量定义在[这里](https://en.wikipedia.org/wiki/Exponent_bias)：
> When interpreting the floating-point number, the bias is subtracted to retrieve the actual exponent.
> * For a single-precision number, the exponent is stored in the range 1 .. 254 (0 and 255 have special meanings), and is interpreted by subtracting the bias for an 8-bit exponent (127) to get an exponent value in the range −126 .. +127.
> * For a double-precision number, the exponent is stored in the range 1 .. 2046 (0 and 2047 have special meanings), and is interpreted by subtracting the bias for an 11-bit exponent (1023) to get an exponent value in the range −1022 .. +1023.
> * For a quad-precision number, the exponent is stored in the range 1 .. 32766 (0 and 32767 have special meanings), and is interpreted by subtracting the bias for a 15-bit exponent (16383) to get an exponent value in the range −16382 .. +16383.

计算举例：
$10000000101_2$ = $1029_{10}$
![](http://latex.codecogs.com/gif.latex?%2410000000101_2%24%20%3D%20%241029_%7B10%7D%24)
e = 1029 - 1023 = 6

##### 小数位
小数部分有52位。**整数部分总是1**，不用存储。

所以有效数字位数共53位：52小数位数 + 1位整数位数。

$log2^{53}\approx15.95$ 
![](http://latex.codecogs.com/gif.latex?%24log2%5E%7B53%7D%5Capprox15.95%24)
所以双精度浮点数可以保证15位十进制有效数。

##### 计算举例
`0 10000000011 0111000000000000000000000000000000000000000000000000`

符号位为0，即+。
指数位e = `10000000011`(1027) - 1023 = 4。
有效数为 1.0111。

即+$1.0111 * 2^4$ = 23
![](http://latex.codecogs.com/gif.latex?&plus;%241.0111%20*%202%5E4%24%20%3D%2023)

规约形式的最小正数是：
`0 00000000001 0000000000000000000000000000000000000000000000000001`
![1E5F8FF4-21B4-46AA-A502-65607F025BDF.png](https://xwcoder.github.io/resources/26BD4F2F544E9ECDCD51DB0F26D72B83.png)

#### 非规约形式
当指数位编码是`00000000000`，并且小数部分不为0时为**非规约形式**。与规约形式相比有两点不同：
1. 指数部分的偏移量比规约形式少1，对双精度浮点格式来说即1022，所以非规约形式的指数e总是-1022。
2. **整数部分为0**。

#### 特殊值

* 指数部分编码为`11111111111`，小数部分为0时，表示正负无穷。
* 指数部分编码为`11111111111`，小数部分不为0时，表示NaN。
* 指数部分和小数部分编码全为0时，表示±0。

```js
0 11111111111 0000000000000000000000000000000000000000000000000000 // +∞
1 11111111111 0000000000000000000000000000000000000000000000000000 // -∞
0 11111111111 0000000000000000000000000000000000000000000000000001 // NaN
0 11111111111 1000000000000000000000000000000000000000000000000001 // NaN
0 00000000000 0000000000000000000000000000000000000000000000000000 // +0
1 00000000000 0000000000000000000000000000000000000000000000000000 // -0
```

![120BD089-B4F9-4BFB-868F-BA8B7C49B1F4.png](https://xwcoder.github.io/resources/17A9BEF22BCE9DA0060397A374B140FC.png)

## Number
这部分主要计算Number类型上定义的几个常量值。

### Number.MAX_VALUE
二进制表示如下，即指数部分和小数部分均取最大值。
`0 11111111110 1111111111111111111111111111111111111111111111111111`

$(2 - 2^{-52}) * 2^{1023}$
![](http://latex.codecogs.com/gif.latex?%24%282%20-%202%5E%7B-52%7D%29%20*%202%5E%7B1023%7D%24)

![291F28FB-E4A7-41F3-817D-9201D6ED0582.png](https://xwcoder.github.io/resources/C15A81A1009E7D8AD9A9772F544DCE42.png)

### Number.MIN_VALUE
在使用IEEE 754-2008双精度浮点格式的实现中，`Number.MIN_VALUE`表示**非规约形式**的最小正数。
> The value of Number.MIN_VALUE is the smallest positive value of the Number type, which is approximately 5 × 10-324.
> In the IEEE 754-2008 double precision binary representation, the smallest possible value is a denormalized number. If an implementation does not support denormalized values, the value of Number.MIN_VALUE must be the smallest non-zero positive value that can actually be represented by the implementation.  -- [ecma262](https://tc39.github.io/ecma262/#sec-number.min_value)

非规约形式下的最小正数：
`0 00000000000 0000000000000000000000000000000000000000000000000001`
$2^{-52} * 2^{-1022}$
![](http://latex.codecogs.com/gif.latex?%242%5E%7B-52%7D%20*%202%5E%7B-1022%7D%24)

![B62F064B-354D-48F6-A076-EF293B1B1110.png](https://xwcoder.github.io/resources/5F701523469D9F5FA2F6DB56B70A3AE4.png)

### Number.MAX_SAFE_INTEGER/Number.MIN_SAFE_INTEGER

`Number.MIN_SAFE_INTEGER = -Number.MAX_SAFE_INTEGER`

由于双精度浮点格式的小数部分只有52位，所以其能准确表示的最大正整数为：符号位为0，小数部分编码全部为1，指数位计算结果为52。即$1.111...111 * 2^{52} = 2^{53} - 1$ ![](http://latex.codecogs.com/gif.latex?%241.111...111%20*%202%5E%7B52%7D%20%3D%202%5E%7B53%7D%20-%201%24)。超过此数值的整数无法使用双精度浮点格式准确表示。

![D0147EA2-EE4A-4CBC-8E65-FAE43E9D9F29.png](https://xwcoder.github.io/resources/62914D8BA7BC6749B34716C46DCC16E2.png)

因此，为了操作安全，数组在一些诸如`concat`, `from`等方法中要判断操作结果的长度是否在安全范围内。
![DB591A73-723C-45B6-B979-D2D6B7A53329.png](https://xwcoder.github.io/resources/F45ED286AB691241B42AB9A932B3ED36.png)

## Smi
在JavaScript引擎性能优化相关的文章中经常会看到Smi(Small Integer)，SMIs(Small Integers)，即小整数。这部分内容简单介绍Smi。

Smi是JavaScript引擎的优化手段，这部分内容主要以v8进行介绍，其他JS引擎也有类似优化。

### 什么是Smi
通过前面的介绍我们知道BigInt出现之前，JavaScript中只有Number一种数值类型，采用IEEE 754双精度浮点格式表示，不区分整型和浮点型。但是在程序中会频繁使用小整数，比如数组的下标和数学运算等。如果总是从堆(heap)中分配内存存储数值并被gc管理，并且进行浮点运算，开销太大而且性能低。所以v8在内部对小整数(Smi)使用了整数格式表示，而不是IEEE 754浮点格式。

这样在v8内部就有两类值：一种是Smi，直接表示一个整数；一种是堆对象，称作HeapObject。
可以参考[src/objects.h](https://github.com/v8/v8/blob/7.6.141/src/objects.h#L36)中的注释：
![8AC174A1-B388-46C5-AFA1-D2B23381B6D4.png](https://xwcoder.github.io/resources/5895A18732597FC085A109294B828B51.png)

![4CC643C5-FA9A-4A9F-9E6C-78037954BB52.png](https://xwcoder.github.io/resources/38570609BC0DADFEDED0A3CD3C0E6C25.png)

![tagging-fc0ad12f99d1473bb558877337917d2c.png](https://xwcoder.github.io/resources/E842442174011892DA2A2C9D16D1C06A.png)

v8通过最低的一个比特位来区分Smi和指向HeapObject的指针：最低比特位为1时是指向HeapObject的指针，为0时是Smi。[include/v8-internal.h](https://github.com/v8/v8/blob/7.6.141/include/v8-internal.h#L38)。

![A21A11AC-ED3E-4786-B8CD-780FA0F800FC.png](https://xwcoder.github.io/resources/9770051D4A36B440FE54CFED41391212.png)

### Smi的范围
在64位系统中，Smi的低32位全部为0，只使用高32位表示数值，所以其取值范围是[$-2^{31}$, $2^{31} - 1$]
![](http://latex.codecogs.com/gif.latex?%5B%24-2%5E%7B31%7D%24%2C%20%242%5E%7B31%7D%20-%201%24%5D)。

在32位系统中，Smi的最低一位为0，使用剩余的31位表示数值，所以其取值范围是[$-2^{30}$, $2^{30} - 1$]
![](http://latex.codecogs.com/gif.latex?%5B%24-2%5E%7B30%7D%24%2C%20%242%5E%7B30%7D%20-%201%24%5D)。

Smi的范围信息也定义在[include/v8-internal.h](https://github.com/v8/v8/blob/7.6.141/include/v8-internal.h#L107)。
![0F5D133E-D5E5-446B-B631-B83DBFC1713D.png](https://xwcoder.github.io/resources/1B0DBCF6A0C6F69EB55C971AD55BC3A5.png)

64位系统中，`kSmiValueSize`是32；32位系统中`kSmiValueSize`是31。

```c
// 以64位系统为例
kSmiMinValue = (static_cast<unsigned int>(-1)) << (kSmiValueSize - 1)
  = 111...111 << (32 - 1)
  = 111...111 << 31 // -1的补码
  = 111...111000...000 // 31个0
  = -2^31
kSmiMaxValue = -(kSmiMinValue + 1) = 2^31 - 1
```

## 参考资料
* [wikipedia: Double-precision floating-point format](https://en.wikipedia.org/wiki/Double-precision_floating-point_format)
* [wikipedia: IEEE 754](https://zh.wikipedia.org/wiki/IEEE_754)
* [ecma 262](https://tc39.github.io/ecma262/#sec-ecmascript-language-types-number-type)
* [An Introduction to Speculative Optimization in V8](https://ponyfoo.com/articles/an-introduction-to-speculative-optimization-in-v8#feedback-lattice), by Meurer
