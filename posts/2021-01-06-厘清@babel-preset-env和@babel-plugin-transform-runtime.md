## Babel和core-js的关系

> Babel is a compiler, core-js is a polyfill.

`Babel`是一个编译器，用来将那些使用了新语法的代码编译成老旧运行环境(比如ie浏览器)可以识别运行的代码，比如将`const`，`let`转换为`var`。

`core-js`是一个polyfill库，使新的api(native functions)在老旧运行环境下可用，比如`Object.assign`, `Promise`。polyfill库有很多比如`core-js`, `es-shims`。

由于历史原因和`core-js`本身的优秀，`Babel`做了很多工作来方便开发者使用`core-js`，比如`@babel/preset-env`的`useBuiltIns`和`corejs`设置项，`@babel/plugin-tranfrom-runtime`的`corejs`设置项。

`Babel`主要做两件事：
- 转换语法。
- 添加polyfill。

## @babel/preset-env
`@babel/preset-env`的设置项`useBuiltIns`用来控制是否添加`core-js` polyfill以及怎么添加。

- `false`: 不添加
- `entry`: 在入口文件添加，需要手动在入口文件添加`import 'core-js'`。缺点是它会import全部polyfill，即使没有用到。
- `usage`: 在当前文件添加，并且只添加用到的polyfill。

举例来看，`iOS 9`支持`Object.assign`，`iOS 8`不支持; 两者都不支持`class`。

```javascript
// babel.config.js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        debug: true,
        useBuiltIns: 'usage',
        corejs: 3,
        modules: false,
        targets: {
          ios: '8'
        },
      }
    ],
  ],
};
```

```javascript
// source
export const config = Object.assign({}, { type: 1 });

export class Person {};
```

```javascript
// out
import "core-js/modules/es.object.assign.js";

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

export var config = Object.assign({}, {
  type: 1
});
export var Person = function Person() {
  _classCallCheck(this, Person);
};
```

当`targets`是`iOS 8`时，`@babel/preset-env`自动导入了`core-js`以支持`Object.assign`，并且使用了一些辅助函数以支持`class`语法。这些辅助函数被称为`helper`。

通过转译输出的代码可以看到，这样的编译有两个缺点：
- polyfill的引入方式会污染全局环境。
- `helper`直接内嵌在文件中，当一个项目有很多文件，各文件中会有大量重复的`helper`，这会造成bundle size变大。

## @babel/plugin-transform-runtime
`@babel/plugin-transform-runtime`主要有两个作用：

一是从`@babel/runtime`包中导入`helper`，而不是直接嵌入到当前输出文件，从而减小包体积。这个能力通过`helper`选项控制，默认是`true`。

二是添加polyfill。通过`corejs`选项控制，默认是`false`，不添加。

根据`corejs`选项值需要引入`@babel/runtime`或`@babel/runtime-corejs2`或`@babel/runtime-corejs3`。

将上例中`babel.config.js`添加`@babel-plugin-transform-runtime`:
```javascript
// babel.config.js
plugins: [
  [
    '@babel/plugin-transform-runtime',
    {
      helpers: true,
      corejs: 3,
    }
  ],
]
```

```javascript
// out
import _classCallCheck from "@babel/runtime-corejs3/helpers/classCallCheck";
import _Object$assign from "@babel/runtime-corejs3/core-js-stable/object/assign";
export var config = _Object$assign({}, {
  type: 1
});
export var Person = function Person() {
  _classCallCheck(this, Person);
};
```

从转译输出可以看到:
- 从`@babel/runtime-xxx`导入了`helper`。
- polyfill的引入方式不污染全局环境。

## @babel/plugin-transform-runtime和@babel/preset-env没有关系

`@babel/plugin-tranform-runtime`和`@babel/preset-env`都可以添加polyfill，但是两者是完全独立的。看一下`Babel`作者的说法：

> useBuiltIns and @babel/plugin-transform-runtime are mutually exclusive. Both are used to add polyfills: the first adds them globally, the second one adds them without attatching them to the global scope.
You should decide which behavior you want and stick with it.

[link](https://github.com/babel/babel/issues/10271#issuecomment-528379505)

`@babel/preset-env`可以根据`targets`导入需要的polyfill，但是会污染全局环境。`@babel/plugin-tranform-runtime`导入polyfill不会污染全局环境，但是它是环境无关的，只要用到的api存于在于`core-js` polyfill中就会被导入。

不要混用两者导入polyfill的能力，两者是独立的。例如将Demo中的targets改成`iOS 9`，编译后的输出依然会导入`Object.assign`的polyfill。`@babel/preset-env`和`@babel/plugin-transform-runtime`导入polyfill各有优缺点，需要根据项目的需求选择使用其一。

当开发类库时多使用`@babel/plugin-tranform-runtime`，可以防止污染全局环境从而造成冲突。

## RFC10008

我们肯定希望使用`@babel/preset-env`时可以不污染全局变量，使用`@babel/plugin-transform-runtime`时也可以设置类似`targets`的目标环境。为此`Babel`作者提了一个[`RFC`](https://github.com/babel/babel/issues/10008)。

将来可能会:
- 将`targets`作为顶级配置，这样所有`preset`和`plugin`都可以共享`targets`。
- 提供顶级配置项`polyfills`。
