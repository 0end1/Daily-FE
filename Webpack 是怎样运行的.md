## Webpack是怎样运行的？

在平时开发中我们经常会用到`Webpack`这个时下最流行的前端打包工具。它打包开发代码，输出能在各种浏览器运行的代码，提升了开发至发布过程的效率。

我们知道一份`Webpack`配置文件主要有入口(entry)、输出文件(output)、模式、加载器(loader)、插件(plugin)等几部分。但如果只需要组织JS文件的话，指定入口和输出文件路径即可完成一个迷你项目的打包。下面我们来通过一个简单的项目来看一下`Webpack`是怎样运行的。

### 同步加载

> 本文使用webpack ^4.30.0作为示例。为了更好地观察产出的文件，我们将模式设置为development关闭代码压缩，再开启source-map支持原始代码调试。除此之外，我们还简单的写了一个插件MyPlugin来去除源码中的注释。

新建`src/index.js`

~~~
console.log('Hello webpack!');
~~~

新建`Webpack`配置文件`webpack.config.js`

~~~
const path = require('path');
const MyPlugin = require('./src/MyPlugin.js')

module.exports = {
  mode: 'development',
  devtool: 'source-map',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist')
  },
  plugins:[
    new MyPlugin()
  ]
};
~~~

新建`src/MyPlugin.js`

~~~
class MyPlugin {
  constructor(options) {
    this.options = options
    this.externalModules = {}
  }

  apply(compiler) {
    var reg = /("([^\\\"]*(\\.)?)*")|('([^\\\']*(\\.)?)*')|(\/{2,}.*?(\r|\n))|(\/\*(\n|.)*?\*\/)|(\/\*\*\*\*\*\*\/)/g
    compiler.hooks.emit.tap('CodeBeautify', (compilation)=> {
      Object.keys(compilation.assets).forEach((data)=> {
        let content = compilation.assets[data].source() // 欲处理的文本
        content = content.replace(reg, function (word) { // 去除注释后的文本
          return /^\/{2,}/.test(word) || /^\/\*!/.test(word) || /^\/\*{3,}\//.test(word) ? "" : word;
        });
        compilation.assets[data] = {
          source(){
            return content
          },
          size(){
            return content.length
          }
        }
      })
    })
  }
}
module.exports = MyPlugin
~~~

现在我们运行命令`webpack --config webpack.config.js`，打包完成后会多出一个输出目录`dist:dist/main.js`。`main`是`webpack`默认设置的输出文件名。我们快速瞄一眼这个文件：

~~~
(function(modules){
  // ...
})({
  "./src/index.js": (function(){
    // ...
  })
});
~~~

整个文件只含一个立即执行函数，我们称它为`webpackBootstrap`，它仅接收一个对象——未加载的模块集合(modules)，这个modules对象的key是一个路径，value是一个函数。你也许会问，这里的模块是什么？他们又是如何加载的呢？
在细看产出代码前，我们先丰富一下源代码：
新文件`src/utils/math.js`:

~~~
export const plus = (a, b) => {
  return a + b;
};
~~~

修改`src/index.js`:

~~~
import { plus } from './utils/math.js';

console.log('Hello webpack!');
console.log('1 + 2: ', plus(1, 2));
~~~

我们按照ES规范的模块化语法写了一个简单的模块src/utils/math.js，给src/index.js引用。Webpack用自己的方式支持了ES6 Module规范，前面提到的module就是和ES6 module对应的概念。

接下来我们看一下这些模块是如何通过ES5代码实现的。再次运行命令`webpack --config webpack.config.js`后查看输出文件：

~~~
(function(modules){
  // ...
})({
  "./src/index.js": (function(){
    // ...
  }),
  "./src/utils/math.js": (function() {
    // ...
  })
});
~~~

IIFE传入的modules对象里多了一个键值对，对应着新模块src/utils/maths.js，这和我们在源代码中拆分的模块互相呼应。然而，有了modules只是第一步，这份文件最终达到的效果应该是让各个模块按开发者编排的顺序运行。

#### 探究webpackBootstrap

接下来看看`webpackBootstrap`函数中有些什么：

~~~
// webpackBootstrap
(function(modules){

  // 缓存 __webpack_require__ 函数加载过的模块
  var installedModules = {};

  /**
   * Webpack 加载函数，用来加载 webpack 定义的模块
   * @param {String} moduleId 模块 ID，一般为模块的源码路径，如 "./src/index.js"
   * @returns {Object} exports 导出对象
   */
  function __webpack_require__(moduleId) {
    // ...
  }

  // 在 __webpack_require__ 函数对象上挂载一些变量及函数 ...

  // 传入表达式的值为 "./src/index.js"
  return __webpack_require__(__webpack_require__.s = "./src/index.js");
})(/* modules */);
~~~

可以看到其实主要做了两件事：

 1. 定义一个模块加载函数`__webpack_require__`
 2. 使用加载函数加载入口模块`./src/index.js`
 
整个webpackBootstrap中只出现了入口模块的影子，那其他模块又是如何加载的呢？我们顺着__webpack_require__("./src/index.js")细看加载函数的内部逻辑：

~~~
function __webpack_require__(moduleId) {
  // 重复加载则利用缓存
  if (installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }

  // 如果是第一次加载，则初始化模块对象，并缓存
  var module = installedModules[moduleId] = {
    i: moduleId,  // 模块 ID
    l: false,     // 模块加载标识
    exports: {}   // 模块导出对象
  };

  /**
    * 执行模块
    * @param module.exports -- 模块导出对象引用，改变模块包裹函数内部的 this 指向
    * @param module -- 当前模块对象引用
    * @param module.exports -- 模块导出对象引用
    * @param __webpack_require__ -- 用于在模块中加载其他模块
    */
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

  // 模块加载标识置为已加载
  module.l = true;

  // 返回当前模块的导出对象引用
  return module.exports;
}
~~~

首先，加载函数使用了闭包变量`installedModules`，用来将已加载过的模块保存在内存中。接着是初始化模块对象，并把它挂载到缓存里。然后是模块的执行过程，加载入口文件时`modules[moduleId]`其实就是`.src/index.js`对应的模块函数。执行模块函数前传入了跟模块相关的几个实参，让模块可以到处内容，以及加载其他模块的导出。最后标识该模块加载完成，返回模块的到处内容。

根据`__webpack_require__`的混村和导出逻辑，我们得知在整个IIFE执行过程中，加载已缓存的模块时，都会直接返回`installedModules[moduleId].exports`，换句话说，相同的模块只有在第一次引用的时候才会执行模块本身。

#### 模块执行函数

`__webpack_require__`中通过`modules[moduleId].call()`执行了模块执行函数，下面我们就进入到`webpackBootstrap`的参数部分，看看模块的执行函数。

~~~
/*** 入口模块 ./src/index.js ***/
"./src/index.js": (function (module, __webpack_exports__, __webpack_require__) {
  "use strict";
// 用于区分 ES 模块和其他模块规范，不影响理解 demo，战略跳过。
  __webpack_require__.r(__webpack_exports__);
  /* harmony import */
 // 源模块代码中，`import {plus} from './utils/math.js';` 语句被 loader 解析转化。
    // 加载 "./src/utils/math.js" 模块，
  var _utils_math_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/utils/math.js");
  console.log('Hello webpack!');
  console.log('1 + 2: ', Object(_utils_math_js__WEBPACK_IMPORTED_MODULE_0__["plus"])(1, 2));
}),

"./src/utils/math.js": (function (module, __webpack_exports__, __webpack_require__) {
  "use strict";
  __webpack_require__.r(__webpack_exports__);
  /* harmony export (binding) */
// 源模块代码中，`export` 语句被 loader 解析转化。
  __webpack_require__.d(__webpack_exports__, "plus", function () {
    return plus;
  });
  const plus = (a, b) => {
    return a + b;
  };
})
~~~

执行顺序是：入口模块->工具模块->入口模块。入口模块首先通过`__webpack_require__("./src/utils/math.js")`拿到了工具模块的`exports`对象。再看工具模块，`ES`导出语法转化成了`__webpack_require__.d(__webpack_exports__, [key], [getter])`,而`__webpack_require__.d`函数的定义在`webpackBootstrap`内：

~~~
// 定义 exports 对象导出的属性。
  __webpack_require__.d = function (exports, name, getter) {

    // 如果 exports （不含原型链上）没有 [name] 属性，定义该属性的 getter。
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, {
        enumerable: true,
        get: getter
      });
    }
  };

  // 包装 Object.prototype.hasOwnProperty 函数。
  __webpack_require__.o = function (object, property) {
    return Object.prototype.hasOwnProperty.call(object, property);
  };
~~~

可见`__webpack_require__.d`其实就是`Object.defineProperty`的简单包装。

引用工具模块导出的变量后，入口模块再执行它剩余的部分。至此，Webpack基本的模块执行过程就结束了。

好了，我们用流程图总结下Webpack模块的加载思路：


### 异步加载