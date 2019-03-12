## babel之配置文件.babelrc入门详解

### 什么是Babel

---
官方解释，是下一代JS语法的编译器。

既然是下一代JS的标准，浏览器因版本问题会对此有兼容性问题，JS新的方法都不能使用，但是目前我们在项目中一直提倡使用最新的语法糖编写，不但能减少代码，而且async,await等新特性还解决了回调的编写问题，减轻了代码的维护成本。

Babel因此而生，它可以让你放心大胆地使用大部分的JS新的标准方法，然后编译成兼容绝大多数主流浏览器的代码。在项目工程脚手架中，一般会使用.babelrc文件通过配置一些参数配合webpack进行打包压缩。再次整理一份资料，详细介绍各配置项的意义所在，以便清晰了解和使用。

```
以下配置主要针对webpack3+写法
```

#### Babel转译器
在.babelrc配置文件中，主要是对预设(presets)和插件(plugins)进行配置，因此不同的转译器作用不同的配置项，大致可分为以下三项：

```
1.语法转译器。主要针对JS最新的语法糖进行编译，并不负责转译JS新增的api和全局对象。例如let/const就可以编译，而includes/Object.assign等并不能被编译。常用到的转译器有，babel-preset-env、babel-preset-es2015、babel-preset-es2016、babel-preset-es2017、babel-preset-latest等。在实际开发中只选用babel-preset-env来代替余下的，但是还需要配上JS的制作规范一起使用，同时也是官方推荐：

{
  "presets": ["env", {
      "modules": false
    }],
    "stage-2"
}

2.补丁转译器。主要负责转译JS新增的api和全局对象，例如babel-plugin-transform-runtime这个插件能够编译Object.assign,同时也可以引入babel-polyfill进一步对includes这类用法保证在浏览器的兼容性。Object.assign会被编译成以下代码：

__WEBPACK_IMPORTED_MODULE_1_babel_runtime_core_js_object_assign___default()

3.jsx和flow插件，这类转译器用来转译JSX语法和移除类型声明，使用React的时候你将用到它，转译器名称是babel-preset-react

```

#### 创建预设(presets)

主要通过npm安装babel-preset--xx插件来配合使用，例如通过npm install babel-preset-stage-2 babel-preset-env --save-dev安装，会有相应如下配置：

```
{
  "presets": [
    ["env", options],
    "stage-2"
  ]
}
```
##### stage-2配置

babel主要提供以下几种转译器包，括号里面是对应配置文件的配置项：

```
babel-preset-stage-0(stage-0) 
babel-preset-stage-1(stage-1) 
babel-preset-stage-2(stage-2) 
babel-preset-stage-3(stage-3)
```
不同阶段的转译器之间是包含的关系，preset-stage-0转译器除了包含了preset-stage-1的所有功能还增加了transform-do-expressions插件和transform-function-bind插件，同样preset-stage-1转译器除了包含preset-stage-2的全部功能外还增加了一些额外的功能。

##### options配置介绍

官方推荐使用babel-preset-env来代替一些插件包的安装(es2015-arrow-functions, es2015-block-scoped-functions等等)，并且有如下几种配置信息，介绍几个常用的：

```
{
    "targets": {
        "chrome": 52,
        "browsers": ["last 2 versions", "safari 7"],
        "node":"6.10"
    }
    "modules": false
}
```
更多配置可以参考官网：https://babeljs.io/docs/plugins/preset-env/

targets可以指定兼容浏览器版本，如果设置了browers，那么就会覆盖targets原本对浏览器的限制配置。

targets.node对node版本进行编译

modules通常都会设置为false，这样就不会对es6的模块语法(import)进行转换了

#### 插件（plugins）

产检配置项同预设配置项一样，需要搭配babel相应的产检进行配置，可以选择配置插件来满足单个需求，例如早期我们会进行如下配置：
```
{
  "plugins": [
    "check-es2015-constants",
    "es2015-arrow-functions",
    "es2015-block-scoped-functions",
    // ...
  ]
}
```
但是这些插件丛书写到维护都极为麻烦，后来官方统一推荐使用env，全部替代了这些单一插件功能，可以简化配置如下，也就是我们前面提到的babel-preset-env:

```
{
  "presets": [
    "es2015"
  ]
}
```

这里主要介绍两款常用插件，分别是babel-plugin-transform-runtime, babel-plugin-syntax-dynamic-import。

基本配置代码如下：

```
{
  "plugins": [
    "syntax-dynamic-import",["transform-runtime"]
  ]
}
```
##### transform-runtime

为了解决这种全局对象或者全局对象方法编译不足的情况，出现了transform-runtime这个插件，但是它只会对es6的语法进行转换，而不会对新api进行转换。如果需要转换新api，也可以通过使用babel-polyfill来规避兼容性问题。

对Object.assign进行编译，配置与未配置经过webpack编译后的代码片段如下：

```
// 未设置代码片段：
__webpack_require__("ez/6");
var aaa = 1;

function fna() {
  var dd = 33333;
  var cc = Object.assign({ key: 2 });
  var xx = String.prototype.repeat.call('b', 3);
  if ("foobar".String.prototype.includes("foo")) {
    var vv = 1;
  }

  return dd;
}
```

```
// 设置代码片段：
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_1_babel_runtime_core_js_object_assign___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_1_babel_runtime_core_js_object_assign__);

__webpack_require__("ez/6");
var aaa = 1;

function fna() {
  var dd = 33333;
  var cc = __WEBPACK_IMPORTED_MODULE_1_babel_runtime_core_js_object_assign___default()({ key: 2 });
  var xx = String.prototype.repeat.call('b', 3);
  if ("foobar".String.prototype.includes("foo")) {
    var vv = 1;
  }

  return dd;
}
```

对class定义类会进行编译，配置与未配置经过webpack编译后的代码片段如下：

```
// 未设置代码片段：
function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var Canvas = function Canvas(height, width) {
  _classCallCheck(this, Canvas);

  this.height = height;
  this.width = width;
};

var Canvas2 = function Canvas2(height, width) {
  _classCallCheck(this, Canvas2);

  this.height = height;
  this.width = width;
};
```

```
// 设置代码片段：
/* harmony import */ var __WEBPACK_IMPORTED_MODULE_0_babel_runtime_helpers_classCallCheck___default = __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0_babel_runtime_helpers_classCallCheck__);

var Canvas = function Canvas(height, width) {
  __WEBPACK_IMPORTED_MODULE_0_babel_runtime_helpers_classCallCheck___default()(this, Canvas);

  this.height = height;
  this.width = width;
};

var Canvas2 = function Canvas2(height, width) {
  __WEBPACK_IMPORTED_MODULE_0_babel_runtime_helpers_classCallCheck___default()(this, Canvas2);

  this.height = height;
  this.width = width;
};
```

##### syntax-dynamic-import

这个插件主要解决动态引入模块的问题

```
function nDate() {
  import('moment').then(function(moment) {
    console.log(moment.default().format());
  }).catch(function(err) {
    console.log('Failed to load moment', err);
  });
}

nDate();
```

```
如果.babelrc配置项中使用了"stage-2"，也可以不使用该插件，同样支持动态模块引入。
```

不然就会报以下错误：

* Module build failed: SyntaxError: 'import' and 'export' may only appear at the top level, or (import 和 export只能在最外层，也就是不能用在函数或者块中)
* Module build failed: SyntaxError: Unexpected token, expected {

#### 其他配置项
##### ignore

主要作用就是可以指定不编译哪些代码

```
{
  "ignore":["./module/a.js"]
}
```

##### minified

主要设置编译后是否是压缩，boolean类型，如果使用babel-cli进行打包压缩编译文件，这个配置能够起到作用，但是目前大部分项目还是会依赖第三方打包工具，例如webpack，所以这个配置参数一般不用设置，webpack中的UglifyJsPlugin做了压缩的工作。

##### comments

再生成的文件中不产生注释，boolean类型，webpack插件中的UglifyPlugin也同样集成了这个功能。

##### env

基本配置如下：

```
{
  "env": {
    // test 是提前设置的环境变量，如果没有设置BABEL_ENV则使用NODE_ENV，如果都没有设置默认就是development
    "test": {
      "presets": ["env", "stage-2"],
      // instanbul是一个用来测试转码后代码的工具
      "plugins": ["istanbul"]
    }
  }
}
```

#### 再谈兼容性问题

Babel默认只转换新的JS语法，而不转换新的API,比如Iterator、Generator、Set、Maps、Promise等等全局对象，以及一些定义在全局对象上的方法（比如Object.assign）都不会转码，具体的可以参考babel-plugin-transform-runtime模块的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)文件。

这里主要涉及到babel编译后依然会存在浏览器兼容性问题，一般会使用transform-runtime和babel-polyfill配合使用，对于后者只需要在项目入口文件require引入即可。

当然在使用类似Object.assign函数功能时，可以使用lodash库来替代，promise可以使用Q.js替代等等方案，这样一来可以不需要引入以上插件，具体可以根据项目具体安排。

#### 总结

.babelrc配置文件主要还是以presets和plugins组成，通过和webpack配合进行使用。

vue项目中配置如下：

```
{
  "presets": [
    ["env", {
      "modules": false
    }],
    "stage-2"
  ],
  // 下面指的是在生成的文件中，不产生注释
  "comments": false,
  "plugins": ["transform-runtime","syntax-dynamic-import"],
  "env": {
    // test 是提前设置的环境变量，如果没有设置BABEL_ENV则使用NODE_ENV，如果都没有设置默认就是development
    "test": {
      "presets": ["env", "stage-2"],
      // instanbul是一个用来测试转码后代码的工具
      "plugins": ["istanbul"]
    }
  }
}
```

react项目中配置如下：

```
{
  "presets": [
    ["env", { "modules": false }],
    "stage-2",
    "react"
  ],
  "plugins": ["transform-runtime"],
  "comments": false,
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": [ "istanbul" ]
    }
  }
}
```

#### 关于stage-x介绍的补充

stage-3包括以下插件：
* transform-async-to-generator 支持async/await
* transform-exponentiation-operator 支持幂运算符语法糖

stage-2包括stage-3的所有插件，额外还包括以下插件：
* syntax-trailing-function-commas 支持尾逗号函数，额...很鸡肋
* transform-object-reset-spread 支持对象的解构赋值

stage-1包括stage-2所有插件，额外还包括以下插件：
* transform-class-constructor-call 支持class的构造函数
* transform-class-properties 支持class的static属性
* transform-decorators 支持es7的装饰者模式即@，这其实是很有用的特性，对于HOC来说这是一个不错的语法糖
* transform-export-extensions 支持export方法

stage-0包括stage-1所有插件，额外还包括以下插件：
* transform-do-expressions 支持在jsx中书写if/else
* transform-function-bind 支持::操作符来切换上下文，类似于es5的bind