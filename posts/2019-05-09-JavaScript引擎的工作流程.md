[TOC]

这篇内容主要翻译整理自以下两个演讲及其对应的文章：

* video: [JavaScript Engines: The Good Parts™ - Mathias Bynens & Benedikt Meurer - JSConf EU 2018](https://www.youtube.com/watch?time_continue=167&v=5nmpokoRaZI)
* video: [A Tale of Types, Classes, and Maps by Benedikt Meurer · JSCamp Barcelona 2018](https://www.youtube.com/watch?v=IFWulQnM5E0)
* post: [JavaScript engine fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)
* post: [JavaScript engine fundamentals: optimizing prototypes](https://mathiasbynens.be/notes/prototypes)

在一个比较高的维度介绍JavaScript引擎的工作流程。

## TL;DR

* JavaScript引擎主要由`parser`、`interpreter`（解释器）、`compiler`（编译器）组成。
* JavaScript引擎的主要工作流程如下： 
  1. `parser`将源代码解析成`AST`(Abstract Syntax Tree, 抽象语法树)。
  2. `interpreter`基于`AST`生成`bytecode`（字节码）后程序就可以开始运行。
  3. 基于`bytecode`运行过程中收集的性能分析数据和一些假设，`compiler`可以将热点`bytecode`编译成执行效率更高的机器码。
  4. 如果某个时刻步骤3中的某些假设失效，则回退到`bytecode`。
* `bytecode`的生成速度快，机器码的生成速度慢。
* 机器码比`bytecode`执行快。
* `compilter`生成的机器码优化程度越高，优化后的机器码占用内存越多。
* JavaScript引擎主要在以下两方面权衡： 
  1. 更快的生成代码 vs 生成更快的代码。
  2. 生成优化程度更高的代码 vs 占用更少的内存。

## 工作流程

![IMAGE](https://xwcoder.github.io/resources/C9251FFE42F901CD548B51B7EA2D4F47.jpg)

JavaScript引擎的工作流程如图所示。

1. `parser`将源代码解析成`AST`(Abstract Syntax Tree, 抽象语法树)。
2. `interpreter`基于`AST`生成`bytecode`（字节码）后程序就可以开始运行。
3. 基于`bytecode`运行过程中收集的性能分析数据和一些假设，`compiler`可以将热点`bytecode`编译成执行效率更高的机器码。
4. 如果某个时刻步骤3中的某些假设失效，则回退到`bytecode`。

### v8

![IMAGE](https://xwcoder.github.io/resources/9666957EC4B11129647EFD4970612B12.jpg)

`v8`中的解释器叫做`Ignition`，编译器叫做`TurboFan`。`Ignition`基于AST生成bytecode并运行，在运行过程中会收集性能分析数据，当一段代码变成热点代码(比如某个函数被频繁调用)，byptecode和性能分析数据会被传递到`TurboFan`，基于性能分析数据`TurboFan`会生成更高效的机器码。

### SpiderMonkey

![IMAGE](https://xwcoder.github.io/resources/A204F675983A502370F1CD3CD8EB5E8A.jpg)

`SpiderMonkey`是Firefox使用的JavaScript引擎。SpiderMonkey有两个优化编译器。`Baseline`会生成一定优化程度的代码(somewhat optimized code)。结合性能分析数据和一些假设，`IonMonkey`会生成更高优化程度的代码。当某些假设失效，回退到`somewhat optimized code`。

### Chakra

![IMAGE](https://xwcoder.github.io/resources/BA9C7B5EAD5A533A64F98D74B28E812F.jpg)

Chakra是之前`Edge`浏览器使用的JavaScript引擎。和`SpiderMonkey`类似，有两个优化编译器。

### JavaScriptCore

![IMAGE](https://xwcoder.github.io/resources/A1FC7B616B35E194C4E4B24D788E57B6.jpg)

JavaScriptCore(JSC)是safari浏览器使用的JavaScript引擎，同时也用在`React Native`中。`JSC`使用三个优化编译器。

## 权衡

不同的JavaScript引擎可能有不同数量的compiler，这主要是出于`更快的生成代码 vs 生成更快的代码`的权衡。

### 更快的生成代码 vs 生成更快的代码

```javascript
let result = 0;
for (let i = 0; i < 4242424242; ++i) {
	result += i;
}
console.log(result);
```

![IMAGE](https://xwcoder.github.io/resources/70C284F9EB5648AA34EC10E75BF42784.jpg)

解释器可以快速生成字节码并运行，但是通常字节码运行效率不高。编译器可以生成运行效率更高的机器码，但是生成机器码需要的时间更长。

![IMAGE](https://xwcoder.github.io/resources/A2EC112E73C59368FEF169823B65A230.jpg)

不同的JavaScript引擎可能会在解释器和编译器之间加入不同的优化层，这主要都是为了在`更快的生成代码`和`生成更快的代码`之间寻找到更好的平衡。

#### v8

![IMAGE](https://xwcoder.github.io/resources/79BA6745ACC3BD12283CB79447287931.jpg)

v8的解释器`Ignition`生成bytecode并运行，在某个时刻引擎判断代码变成了`热点代码`，并开始启动`TurboFan frontend`。TurboFan frontend是TurboFan的一部分，用来处理性能分析数据并构造出初步的机器码。之后初步的机器码会被发送到`TurboFan optimizer`以进行进一步的优化，`TurboFan optimizer`运行在另一个线程。这样在优化的同时，`Ignition`可以继续运行字节码，当优化完毕后，开始运行机器码。

#### SpiderMonkey

![IMAGE](https://xwcoder.github.io/resources/B646A1EBEE6545DACC2DCB852FA209C7.jpg)

SpiderMonkey加入了一个中间的优化编译器`Baseline`。开始解释器会运行字节码，然后热点代码会被发送到`Baseline`，Baseline compiler会生成Baseline code并运行，这一切都在主线程完成。
Baseline code运行一段时间后，SpiderMonkey会启动IonMonkey frontend，这之后的流程类似v8，IonMonkey optimizer在另外的线程进行优化工作，Baseline code继续在主线程运行，优化完毕后开始运行机器码。

#### Chakra

![IMAGE](https://xwcoder.github.io/resources/736FE970A287D118594406302D81AC0E.jpg)

Chakra的架构和SpiderMonkey类似。Chakra在并发上做了更多的工作以避免优化时阻塞主线程。Chakra会将热点代码和性能分析数据拷贝到一个专用进程进行优化，主线程不做任何优化工作。这种方法可能会遗漏优化需要的某些信息，影响优化的质量。

#### JavaScriptCore

![IMAGE](https://xwcoder.github.io/resources/03601F1DF586359F21843F48A98EE2A0.jpg)

JavaScriptCore的所有编译器都是并行执行的，并且没有拷贝操作。主线程仅仅只是触发编译任务。这种方式的优点是不会阻塞主线程，缺点就是多线程会带来复杂度以及各种锁的开销。

### 内存

优化程度越高占用内存越多。

例如一个简单的加法运算函数：

```javascript
function add(x, y) {
	return x + y;
}

add(1, 2);

```

在v8中生成的字节码可能如下：

```
StackCheck
Ldar a1
Add a0, [0]
Return

```

`TurboFan`生成的机器码可能如下：

```
leaq rcx,[rip+0x0]
movq rcx,[rcx-0x37]
testb [rcx+0xf],0x1
jnz CompileLazyDeoptimizedCode
push rbp
movq rbp,rsp
push rsi
push rdi
cmpq rsp,[r13+0xe88]
jna StackOverflow
movq rax,[rbp+0x18]
test al,0x1
jnz Deoptimize
movq rbx,[rbp+0x10]
testb rbx,0x1
jnz Deoptimize
movq rdx,rbx
shrq rdx, 32
movq rcx,rax
shrq rcx, 32
addl rdx,rcx
jo Deoptimize
shlq rdx, 32
movq rax,rdx
movq rsp,rbp
pop rbp
ret 0x18

```

相比字节码，机器码会占用更多内存。
 JavaScript引擎也会在`优化程度更高`和`占用内存更少`之间权衡。
