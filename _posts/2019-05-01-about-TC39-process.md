# TC39
`TC39`是推动`ECMAScript`发展的委员会，成员主要来自各主流浏览器厂商，全称是**Technical Committee 39**。

TC39采用协商一致的工作方式：每项决议需要大部分成员同意并且没有成员强烈反对才可以通过。

## TC39 process
### ES2015(ES6)发布周期过长
由于`ES6`的内容太多，因此发布周期很长，从2009年12月发布`ES5`到2015年7月发布`ES6`经过了差不多6年时间。如此长的周期显然不能适应现代浏览器的迭代速度。

为了解决这个问题，从ES2016开始使用一个叫做`TC39 process`的流程。

### 解决方案: TC39 process
ES的每项特性提案都要经过一个周期，这个周期分为5个阶段，从`Stage 0`到`Stage 4`。ES每年会发布一个版本，包括当年达到stage 4的全部特性。

#### Stage 0: strawman
可以自由提交推动ECMAScript发展的想法。提交者必须是TC39成员或者TC39的注册贡献者。

TC39会议必须要评估提交的文档。[stage-0-proposals](https://github.com/tc39/proposals/blob/master/stage-0-proposals.md)。

#### Stage 1: proposal
特性的正式提案。

此阶段的要求如下：
* 确定一个**主负责人**来主要负责提案，**主负责人**或者**联合主负责人**必须是TC39成员。
* 必须描述清楚要解决的问题。
* 必须要通过例子，API和语义及算法讨论来描述解决方案。
* 必须要指出潜在问题。
* 需要polyfills和demos。

#### Stage 2: draft
可能被纳入规范的第一个版本。此阶段的提案很大可能最终会纳入到规范中。

此阶段的要求如下：
* 提案必须要包含特性的正式语法、语义。
* 提案的描述要尽可能完整，允许存在TODO和占位符。
* 必须包含两个实验性质的实现，其中一个实现可以是转译器，例如Babel。

#### Stage 3: candidate
提案接近完成。此阶段需要实现者和使用者更进一步的反馈。

此阶段的要求如下：
* 规范必须是完整的。
* 评审人和ECMAScript编辑需要在规范文档上签字，评审人需要TC39指定，而不是由**主负责人**指定。
* 至少要有两个符合规范的实现。

#### Stage 4: finished
完成阶段。该特性最终会出现在当年年度发布的ECMAScript规范中。

此阶段的要求如下：
* 必须要通过[Test 262](https://github.com/tc39/test262)的验收测试。
* 要有两个通过测试的实现。
* 规范的实现需要有重要的、有意义使用案例，即被使用的经验。
* ECMAScript编辑需要在规范上签字。

## 参考/相关链接
* [The TC39 process for ECMAScript features](http://2ality.com/2015/11/tc39-process.html)
* [ecma262 repo](https://github.com/tc39/ecma262)
* [proposals repo](https://github.com/tc39/proposals)
