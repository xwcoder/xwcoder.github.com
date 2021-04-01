[TOC]

这篇内容主要翻译整理自以下两个演讲及其对应的文章：

* video: [JavaScript Engines: The Good Parts™ - Mathias Bynens & Benedikt Meurer - JSConf EU 2018](https://www.youtube.com/watch?time_continue=167&v=5nmpokoRaZI)
* video: [A Tale of Types, Classes, and Maps by Benedikt Meurer · JSCamp Barcelona 2018](https://www.youtube.com/watch?v=IFWulQnM5E0)
* post: [JavaScript engine fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)
* post: [JavaScript engine fundamentals: optimizing prototypes](https://mathiasbynens.be/notes/prototypes)

尝试在一个较高的维度介绍JavaScript引擎内部如何表示JS对象，以及常用优化手段之一Inlin Caches(ics)。

## 问题

ECMAScript定义object本质上是字典，key是`string`或`Symbol`。

我们知道，对象的属性也有一些[特性](https://tc39.github.io/ecma262/#sec-property-attributes)，比如`[[Writable]]`, `[[Enumerable]]`等，通过`Object.defineProperty`可以设置这些特性。

```javascript
Object.defineProperty(obj, 'key', {
  enumerable: false,  
  configurable: false,
  writable: false,
  value: 'static'
});

```
![IMAGE](resources/0EFFBA54654A0702868835CA094BE6FC.jpg)

### 内存

应用程序中经常会构造大量具有相同结构的对象，比如表示平面上坐标点的对象`const p1 = {x: 5, y: 6}`，`const p2 = {x: 10, y: 10}`。对于具有相同结构的对象，其同名属性的`[[Writable]]`, `[[Enumerable]]`, `[[Configurable]]`特性信息都是相同的，只有`[[Value]]`不同，如果这些特性信息都单独存储在每一个`JSObject`上，会造成大量内存浪费。**问题1：这些objects能不能共享properties的信息，从而节省内存？**

### 属性访问速度

如图所示，当访问属性`y`，需要在object查找`y`，然后访问`y`的`Property attributes`，最后在`attributes`中获取`[[Value]]`并返回。字典查找的速度比较慢，而属性访问又是程序中频繁使用的操作，**问题2：有没有办法可以加快属性的访问速度？**

## Shape/HiddenClass

### 什么是Shape

先看**问题1**，很容易想到，我们可以**把相同的信息单独存储一份，将`[[Value]]`的值存储在`JSObject`中**，这样就可以达到优化内存的目的。JavaScript引擎也是这样处理的。单独存储的结构信息称为`Shape`，它是对object的描述。`Shape`还有一个更为熟知的名字：`HiddenClass`。不同引擎中它会被称为不同的名字，但都指的是同一种优化手段：

* 学术论文中被称为`HiddenClass`。
* 在v8中被称为`Mpas`，（在这篇[问题](https://github.com/xwcoder/xwcoder.github.com/issues/20)中的map属性）。
* 在Chakra中被称为`Types`。
* 在JavaScriptCore中被称为`Structures`。
* 在SpiderMonkey中被称为`Shapes`。

原演讲中使用`Shape`，所以这篇内容也使用`Shape`这个名字。

![IMAGE](resources/DCC4A52C9934B80DA99022407FC0FD33.jpg)

`Shape`中保存属性名和对应的信息，但是不保存`[[Value]]`，`[[Value]]`存储在`JSObject`中。属性信息中会保存一个特殊值`offset`，它是对应`[[Value]]`在`JSObject`中的偏移量，如果`JSObject`中使用数组保存属性值，那么`offset`就是数组下标。

这样具有相同结构的object就可以共享一份`Shape`。

![IMAGE](resources/1A3D8AACD512EEBCB68474A6EACD036E.jpg)

### 转换链/转换树(Transition chains and trees) 

#### 转换链(Transition chains)

**当在object上添加、删除属性时，其`Shape`也会改变**。从一种Shape转换到另一种Shape，旧Shape保留对新Shape的引用，从而形成了转换链(transition chains)。

```javascript
const object = {}
object.x = 5
object.y = 6
```

![IMAGE](resources/021904038D0EA0265FCC9014E85A1550.jpg)

开始object是空对象，所以它指向一个空shape。之后在object上添加了属性`x`，所以它指向了一个新的shape，新shape包含属性`x`及其信息，`x`的值5存储在JSObject中offset为`0`的位置。最后在object上添加了属性`y`，JSObject又指向了一个新的shape，新shape包含属性`x`和`y`及其信息，`y`的值6存储在JSObject中offset为`1`的位置。

**从以上转换过程可以看到，shape和属性添加的顺序相关，所以{x: 5, y: 6}和{y: 6: x: 5}具有不同的shape。**

每个Shape可以只存储引入的新属性，而不必存储全部属性。每个Shape只需要添加对上一个Shape的引用，就可以访问到全部属性信息。本例中，最后一个Shape只存储属性`y`的信息。

![IMAGE](resources/FFDDE8619BEA97C7C304AA7999070068.jpg)

#### 转换树(Transition trees)

```javascript
const object1 = {}
object1.x = 5
const object2 = {}
object2.y = 6
```

本例中object1和object2会共享同一个空shape，之后由于添加不同属性，分别转换到不同的shape，形成树形结构。

![IMAGE](resources/E93317A89DD5658A44822C9D28430530.jpg)

并不是所有对象都从空Shape开始。对于定义时就包含属性的对象，其Shape从非空Shape开始。对于如下示例：

```javascript
const a = {};
a.x = 6;
const b = { x: 6 };
```

对象a从空Shape开始，之后添加属性`x`，其Shpe转换到包含`x`的Shape。对象b定义时就包含属性x，所以其Shape从包含`x`的Shape开始。

![IMAGE](resources/B00BB164385DFE01D50878AAC3D2AF48.jpg)

考虑下面的例子：
```javascript
const point = {}
point.x = 4
point.y = 5
point.z = 6
```
除了空Shape外，JS引擎会创建三个Shape。

![IMAGE](resources/345991CDC230FDF81B4A2C1A498D6870.jpg)

当我们通过`point.x`访问属性`x`时需要遍历Shape链，时间复杂度是`O(n)`。为了提高查找速度，JavaScript引擎又引入了被称为`ShapeTable`的字典结构，key是属性名，value是属性对应的Shape。开头提到过字典的查找速度也比较慢。

![IMAGE](resources/C73BD6FEE395956A77B8FD489C5EBC75.jpg)

JS引擎使用一种被称为Inline Caches(ICs)的技术手段来优化访问速度。

## Inline Caches

ICs是优化js执行速度的关键手段，也是引入`Shape`的主要目的，其原理是记录下属性的位置和Shape信息，从而减少之后运行时的查找次数。

考虑如下代码:
```javascript
function getX(o) {
	return o.x
}
```

在`JavaScriptCore`中，会生成如下字节码:
![IMAGE](resources/15048E7584203BBA26ECE2C3A229FF1D.jpg)

第一条指令是`get_by_id`，它会从第一个参数`arg1`中获取属性`x`并存储在`loc0`，第二条执行返回`loc0`存储的值。

如图，指令`get_by_id`后有两个插槽(slot)，即ICs。

![IMAGE](resources/A8CD191FAD7E6E8AFBD12271904FF0A3.jpg)

假设调用`getX({ x: 'a' })`。我们知道，对象`{ x: 'a' }`会有一个关联`Shape`，`Shape`上存储了属性`x`的特性和`offset`信息。

当第一次执行`getX({ x: 'a' })`时，`get_by_id`会查找属性`x`，并将Shape和`offset`记录在插槽中。
![IMAGE](resources/F1C8C89AFF89026DFD373821559AA070.jpg)

之后再调用`getX`时，ICs只需要比较参数的Shape和之前记录的Shape是否是同一个，如果相同，那么就可以通过记录的`offset`值访问属性值，省去了耗时的查找过程。

![IMAGE](resources/301C0F8BF0ED3CBA62AE45400706BF90.jpg)

如果以不同对象作为参数调用`getX`就会使IC优化失效。例如执行`getX({ x: 'x', y: 'y' })`记录了`Shape`和`Offset`，再以`{ y: 'y', x: 'x' }`调用则会使IC失效，因为`{ x: 'x', y: 'y' }`和`{ y: 'y', x: 'x' }`具有不同的`Shape`。

**编码时，对于拥有相同结构的对象，尽量保证属性添加的顺序也一致。**

## prototype属性访问优化

前面介绍了`Shape`和Inline Caches，这部分介绍`prototype`上的属性访问优化。

JavaScript是基于原型(prototype)的语言，ES6添加的`class`语法可以看做是prototype的语法糖。

如下代码是等价的：
```javascript
class Bar {
	constructor(x) {
		this.x = x
	}
	getX() {
		return this.x
	}
}
```
```javascript
function Bar(x) {
	this.x = x
}

Bar.prototype.getX = function getX() {
	return this.x;
}
```

prototype也是JavaScript对象，实例通过prototype共享属性和方法。

```javascript
const foo = new Bar(true)
```
通过`new`实例化得到对象foo，foo拥有自己的Shape，Shape上有属性`x`的信息。foo的prototype是`Bar.prototype`，`Bar.prototype`拥也有自己的Shape，Shape上有属性`getX`的信息。`Bar.prototype`也是JavaScript对象，其prototype是`Object.prototype`。`Object.prototype`是原型链的根，所以其prototype是`null`。

![IMAGE](resources/2EAF21F42060CFE600100BE5EDEA8E77.jpg)

创建另外一个实例`qux`，`foo`和`qux`拥有共同的Shape，并且都有到`Bar.prototype`的引用。

![IMAGE](resources/79FD01840A2E59CAE61996957CB8B3BF.jpg)

```javascript
const x = foo.getX()
```
调用`foo.getX()`可以看做是两步操作。第一步获取`getX`；第二步以实例foo作为`this`调用`getX()`。
```javascript
const $getX = foo.getX
const x = $getX.call(foo)
```

我们主要关注第一步的查找过程。

![IMAGE](resources/FA5FB60AF5E27223070881738094D1DB.jpg)

引擎从foo实例开始，首先在foo实例的`Shape`上判断没有属性`getX`，然后沿原型链向上查找，来到`Bar.prototype`，在`Bar.prototype`的`Shape`上查找到属性`getX`的信息，其值的`offset`是`0`，最终获取到了`getX`。

从一个例子开始，看看`prototype`属性访问的时间复杂度和优化手段。
```javascript
class Bar {
	constructor(x) { this.x = x; }
	getX() { return this.x; }
}

const foo = new Bar(true);
const x = foo.getX();
//        ^^^^^^^^^^
```
![IMAGE](resources/43ED32335E36AAE63BE64DE9C2200CBB.jpg)

前面`ICs`部分，为了快速获取属性引擎需要做一次判断：判断IC记录的`Shape`和参数的`Shape`是否为同一个。现在来看下访问prototype上的属性需要做几次判断：
1. 实例`foo`的`Shape`没有改变并且没有`getX`属性。
2. `foo`的`prototype`依然是`Bar.prototype`。
3. `Bar.prototype`的`Shape`没有改变并且有`getX`属性。

由于JS的灵活性，很容使用`Object.setPrototypeOf()`等方式改变object的`prototype`，所第2步的判断是必要的。

那么一般获取`prototype`上的属性要经过`1+2N`次判断。N是属性所在`prototype`的深度。

### 优化手段一：**将实例的prototype引用存储在实例的Shape，而不是实例本身。**
![IMAGE](resources/3A5A6AA174E9DCE109FDD170230FC146.jpg)

这样只需要判断实例的`Shape`没有改变，那么也即可以保证实例的`prototype`没有改变，判断次数由`1+2N`降低为`1+N`。

### 优化手段二：Validity cells

虽然通过`优化手段一`降低了判断次数，但是时间复杂度还是线性的`O(n)`。为了使时间复杂度降低为`常数`，引擎引入了`Validity cells`。
![IMAGE](resources/F6FC63E85C5FB2E02E596C9B74232F42.jpg)

* 每个prototype的Shape都有一个`ValidityCell`与其关联。
* 每当与`ValidityCell`关联的prototype**或其之上的prototype**改变，那么`ValidityCell`失效。

这样提升`prototype`属性访问速度的Inline Cache就保存4个字段。
![IMAGE](resources/CA5EAE00808BE54E048C314DB3A3906A.jpg)

1. `Shape`: 实例的Shape。
2. `Offset`: 属性值的位置索引。
3. `prototype`: 包含属性值的prototype。
4. `ValidityCell`: 与实例Shape指向的prototype关联的`ValidityCell`。

Inline Cache构建起来后，下次访问只需要判断IC保存的`Shape`无变化、`ValidityCell`有效，就可以直接在`prototype`的`Offset`位置读取属性值。

通常来说我的业务代码继承层级不会太深，不过有一个继承层次较深的常用场景是DOM。
```javascript
const anchor = document.createElement('a');
const title = anchor.getAttribute('title')
```
![IMAGE](resources/AAF45429E3EDD4097822B774D6BD0CD8.jpg)

`getAttribute()`在`Element.prototype`上。

## 编码原则
通过以上内容，在编码时要注意以下几点：
1. 对于拥有相同结构的对象，尽量保证属性添加的顺序也一致。
2. 不要随意改动原型链，这会使当前的IC失效。
3. 尽量避免使用`delete`, `delete`会改变实例`Shape`或其原型链，从而使当前的IC失效。
