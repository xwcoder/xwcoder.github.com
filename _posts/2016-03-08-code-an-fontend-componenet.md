# 开发一个前端组件

以HTML5播放器为例，介绍开发前端组件的一些思路和需要考虑的一些问题。这里说的是简单的基于OO的组件，不是某框架的组件，比如ExtJS组件、React组件或者其他`MVC`, `MVVM`框架的组件等。
## 面向对象

`面向对象`仍然是理解事物，抽象问题模型的优秀方式。
### 声明播放器类

``` js
class Player {
  constructor (selector, config) {

  }

  //other methods
}
```

以上是基于ES2015的类声明方式，可以通过`Babel`编译成ES5。如果组件需要兼容到ES5以下，可以使用构造函数的方式:

``` js
function Player (selector, config) {

}

extend(Player.prototype, {
  //other methods
});
```
### 实现继承

在ES2015中实现继承非常容易，使用`extends`关键字即可，同样可以使用`babel`编译到ES5。ES2015的`class`同样是基于`原型(prototype)`的，语法糖而已。

兼容ES5以下的继承实现方式有很多种，比较常用的是`寄生组合式继承`:

``` js
function inherit (sb, sp, overide) {
    var F = function () {};
    F.prototype = sp.prototype;
    sb.prototype = new F();
    sb.prototype.constructor = sb;

    sb.prototype.superClass = sp.prototype;
    sb.superClass = sp.prototype;

    overide = overide || {};
    extend(sb.prototype, overide);

    if (sp.prototype.constructor === Object.prototype.constructor) {
        sp.prototype.constructor = sp;
    }
};
```
## 使用pub/sub方式进行消息通信

基于`pub/sub`的消息通信是组件解耦的优秀方式。类似于`Node.js`, 我们需要一个`EventEmitter`类，[这里](https://github.com/xwcoder/EventEmitter)有一个简单实现，很容易将其改造成符合ES2015的模块系统。
### 继承EventEmitter

`Player`需要继承`EventEmitter`。

``` js
//ES2015语法方式
class Player extends EventEmitter {
  constructor (selector, config) {
    super()
  }

  //other methods
}

//兼容方式
function Player (selector, config) {
    this.superClass.constructor.call(this);
}
inherit(Player, EventEmitter, {
  //other methods
});
```

当`Player`实例的状态发生变化时就可以pub消息的方式进行通知，比如暂停`this.emit('pause')`。任何需要获取`Player`实例状态变化的组件都可以订阅消息，`player.on('pause', fn)`, `player.on('ended', fn)`。
## 模块组合方式

以模块组合的方式完成功能，以`pub/sub`的方式进行解耦。比如`UI`模块, `UI`作为独立的模块存在，提高可扩展性，方便换肤和为点播、直播提供不同的UI。UI模块监听`Player`实例的状体变化，并在界面上体现，比如暂停时按钮状体的变化，加载数据时显示`loading`等。

``` js
class VodUI {
  constructor (player, config) {
    this.player = player;
    this.observerPlayer(); 
  }  

  observerPlayer () {
    this.player.on('puase', data => this.onPause());
    this.player.on('waiting', data => this.onWaiting());
    //...
  }
}
```

可以在`Player`的构造函数中实例化`VodUI`, 或者其他合适的时机。

``` js
class Player extends EventEmitter {
  constructor (selector, config) {
    super()

    this.ui = new VodUI(this);
  }
  //other methods
}
```

其他如剧集列表、推荐、统计等模块类似。
## 异步使用Promise

基于`callback`的异步回调方式有很多弊端，比如`回调金字塔`, `高耦合`, `代码阅读性差`等。`pub/sub`, `Promise`都是解决此类问题的优秀方案，`pub/sub`更适合组件通信, 适合描述组件状态的变化，`Promise`更适合类似`ajax`请求的异步操作。

在浏览器中使用`Promise`需要面对兼容性问题，[这里](https://github.com/xwcoder/promise)有一个`Promise`的简单实现。`jQuery`和`Zepto`的`ajax`模块使用的`Deffered`，[这里](https://github.com/xwcoder/ajax)是使用`Promise`改写的Zepto的ajax模块。`优先面向标准`。

可以约定所有可能的异步操作都使用`Promise`, 不限于`ajax`请求。全面放弃`callback`形式。

``` js
function loadData (vid) {
    return ajax(url)
      .then(({data = {}}) => data, () => {code: 404});
}
```
### 组合多异步任务

有很多类库用来解决多异步任务的`并行执行`，`串行执行`, `任务依赖`，比如[`async`](https://github.com/caolan/async)。`但是在能解决问题的前提下，优先使用标准。`

基于`Promise`设计全部异步操作, 使用`promise.then()`很容易解决`串行执行`和`任务依赖`，使用`promise.all()`很容易组合多任务`并行执行`。

``` js
var videoPm = loadData(vid);
var copyrightPm = videoPm.then(({data = {}}) => checkCopyright(data.data));
var channelPm = loadChannelInfo();

Promise.all([videoPm, copyrightPm, channelPm])
  .then(([videoRet, copyrightRet, channelRet]) => {
    //logic
  });
```
## 实例化

可以直接暴露`Player`类。不过为了避免使用`new`和预留其他实例化操作时机，更好的方式是提供实例化方法。

``` js
function createPlayer (selector, config) {
    return new Player(selector, config);
}
```
## 暴露API

组件需要对外暴露可使用的API和消息。由于`javascript`的语言特性，没有真正意义上的私有方法。那么对于组件API的约束通常有以下几种方式: `局部函数`, `方法命名约定`, `代理对象`。
### 局部函数

组件方法不直接定义在组件类上，而是作为`局部函数`存在。比如获取视频的版权信息, `getCopyright`作为模块内部的`局部函数`存在，可以避免组件外调用。

``` js
function getCopyright () {/** some code **/}

class Player extends EventEmitter {
  doPlay () {
    var copyrightInfo = getCopyright();
  }
}
```

这种方式会影响对事物的抽象，而且会影响组件内部各模块的信息传递，增加代码复杂度。所以不推荐大范围使用。
### 方法命名约定方式

按惯例，以`_`开头的都认为是私有方法，组件并不保证此方法在迭代中会一直保留和返回预期的结果。这种方式只是形式上的约定，并不能强制此方法不能被外部调用。此方式使用非常广泛。
### 代理对象方式

这里说的代理对象不是ES2015的`Proxy`，只是最简单的字面量对象，可以看做`Player`实例的包装，只暴露公共API和消息。这也是不直接暴露`Player`，而是提供`createPlayer`方法的一个原因。

``` js
const methods = ['init', 'play', 'pause', 'seek', 'fullscreen'];
const msg = ['waiting', 'playing', 'pause'];

function createPlayer (selector, config) {
    var player = Player(selector, config),
        proxy = {};

    for (var method of methods) {
      proxy[method] = function (...args) {
        player[method](...args);
      };
    }
    proxy.on = function (type, handler, one) {
        if (msg.indexOf(type) != -1) {
            player.on(type, handler, one);
        }
    }

    proxy.one = function (type, handler) {
        if (msg.indexOf(type) != -1) {
            player.one(type, handler);
        }
    }

    proxy.off = function (type, handler) {
        if (msg.indexOf(type) != -1) {
            player.off(type, handler);
        }
    }

    return proxy;
}
```

如果组件仅在团队内部使用，可以使用`方法命名约定`的方式。如果需要提供给团队外部使用，尤其是提供给第三方使用，建议使用`代理对象`方式。

--- the end ---
