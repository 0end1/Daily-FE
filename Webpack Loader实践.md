## 手把手教你撸一个 Webpack Loader

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/webpack%20loader.jpg)

经常逛webpack官网的同学应该会很眼熟上面的图。正如他宣传的一样，webpack能把左侧各种类型的文件(webpack把他们叫做"模块")统一打包为右边被通用浏览器支持的文件。webpack就像是魔术师的帽子，放进去一条丝巾，变出来一只白鸽。那这个“魔术”的过程是如何实现的呢？今天我们从webpack的核心概念之一: loader来寻找答案，并着手实现这个“魔术”。看完本文，你可以：

* 知道webpack loader的作用和原理
* 自己开发贴合业务需求的loader

### 什么是Loader?

在撸一个loader之前，我们需要先知道它到底是什么。本质上说，loader就是一个Node模块，这很符合webpack中“万物皆模块”的思路。既然是node模块，那就一定会导出什么。在webpack的定义中，Loader导出一个函数，loader会在转换源模块(resource)的时候调用该函数。在这个函数内部，我们可以通过传入this上下文给Loader API来使用他们。回顾一下头图左边的那些模块，他们就是所谓的源模块，会被loader转化为右边的通用文件，因此我们可以概括下loader的功能：把源模块转换成通用模块。

### Loader怎么用？

知道它的强大功能后，我们要怎么使用loader呢？

#### 1.配置webpack config文件

既然loader是webpack模块，如果我们要使其生效，肯定离不开配置。我们这里收集了三种配置方式，任你挑选

##### 单个loader的配置

增加config.module.rules数组中的规则对象(rule object)。

~~~
let webpackConfig = {
    //...
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                //这里写 loader 的路径
                loader: path.resolve(__dirname, 'loaders/a-loader.js'), 
                options: {/* ... */}
            }]
        }]
    }
}
~~~

##### 多个loader的配置

增加config.module.rules数组中的规则对象以及config.resolveloader。

~~~
let webpackConfig = {
    //...
    module: {
        rules: [{
            test: /\.js$/,
            use: [{
                //这里写 loader 名即可
                loader: 'a-loader', 
                options: {/* ... */}
            }, {
                loader: 'b-loader', 
                options: {/* ... */}
            }]
        }]
    },
    resolveLoader: {
        // 告诉 webpack 该去那个目录下找 loader 模块
        modules: ['node_modules', path.resolve(__dirname, 'loaders')]
    }
}
~~~

##### 其他配置

也可以通过npm link连接到你的项目里，这个方式类似node CLI工具开发，非loader模块专用，本文就不多讨论了。

#### 2.简单上手

配置完成后，但你在webpack项目中引入模块时,匹配到rule就会启用对应的Loader (例如上面的 a-loader 和 b-loader)。这时，假设我们是a-loader的开发者， a-loader会导出一个函数，这个函数接受的唯一参数是一个包含原文件内容的字符串。我们暂且称他为[source]。

接着我们在函数中处理source的转化，最终返回处理好的值。当然返回值的数量和返回方式依据a-loader的需求来定。一般情况下可以通过return返回一个值，也就是转化后的值。如果需要返回多个参数，则需调用`this.callback(err, values...)`来返回。在异步loader中你可以通过抛错来处理异常情况。Webpack建议我们返回1-2个参数，第一个参数是转化后的source，可以是str或buffer。第二个参数可选，是用来当做SourceMap的对象。

#### 3.进阶使用

通常我们处理一类原文件的时候，单一的Loader是不够的。一般我们会将多个Loader串联使用，类似工厂流水线，一个位置的工人质感一种类型的活。既然是串联，那肯定有顺序问题，webpack规定use数组中Loader的执行顺序是从最后一个到第一个，他们符合下面这些规则：

* 顺序最后的Loader第一个被调用，它拿到的参数是source内容
* 书序第一的loader最后被调用，webpack期望它返回JS代码，source map如前面所说是可选的返回值。
* 加在中间的loader被链式调用，他们拿到上个loader的返回值，为下一个loader提供输入。

我们举个例子:

webpack.config.js

~~~
    {
        test: /\.js/,
        use: [
            'bar-loader',
            'mid-loader',
            'foo-loader'
        ]
    }
~~~

在上面的配置中:

* loader的调用顺序是foo-loader --> mid-loader --> bar-loader。
* foo-loader拿到source，处理后把JS代码传递给mid，mid拿到foo处理过的"source"，再处理之后给bar,bar处理完后再交给webpack。
* bar-loader最终把返回值和source map传给webpack。

### 用正确的姿势开发Loader

#### 1.单一职责
#### 2.链式组合
#### 3.模块化
#### 4.无状态
#### 5.使用loader实用工具

请好好利用`loader-utils`包，它提供了许多有用的工具，最常用的就是获取传入loader的options。除了`loader-tuils`之外还有`schema-utils`包。我们可以用`schema-utils`提供的工具，获取用于校验options的JSON Schema常量，从而校验loader options。下面给出的例子简要的结合了上面提到的两个工具包：

~~~
import { getOptions } from 'loader-utils';
import { validateOptions } from 'schema-utils';

const schema = {
  type: object,
  properties: {
    test: {
      type: string
    }
  }
}

export default function(source) {
    const options = getOptions(this);

    validateOptions(schema, options, 'Example Loader');

    // 在这里写转换 source 的逻辑 ...
    return `export default ${ JSON.stringify(source) }`;
};

~~~

#### 6.模块依赖

不同的模块会有不同的依赖形式。比如CSS中我们使用@import和url(...)声明来完成指定，而我们应该让模块系统解析这些依赖。

如何让模块系统解析不同生命方式的依赖呢？下面有两种方法：
* 把不同的依赖声明统一转化为`require`声明。
* 通过`this.resolve`函数来解析路径

对于第一种方式，有一个很好的例子就是css-loader。它把`@import`声明转化为`require`样式表文件，把`url(...)`声明转化为`require`被引用文件。

而对于第二种方式，则需要参考一下`less-loader`。由于要追踪less中的变量和mixin，我们需要把所有的`.less`文件一次编译完毕，所以不能把每个`@import`转为`require`。因此，`less-loader`用自定义路径解析逻辑拓展了less编译器。这种方式运用了我们刚才提到的第二种方式--`this.resolve`通过webpack来解析依赖。

> 如果某种语言只支持相对路径（例如 url(file) 指向 ./file）。你可以用 ~ 将相对路径指向某个已经安装好的目录（例如 node_modules）下，因此，拿 url 举例，它看起来会变成这样：url(~some-library/image.jpg)。

#### 7.代码公用

避免在多个loader里面初始化同样的代码，请把这些公用代码提取到一个运行时的文件里，然后通过`require`把他引进每个loader。

#### 8.绝对路径

不要在loader模块里写绝对路径，因为当项目路径变了，这些路径会干扰webpack计算Hash(把module路径转化为module的引用id)。loader-utils里有一个`stringiFyRequest`方法，它可以把绝对路径转化为相对路径。

#### 9.同伴依赖

如果你开发的loader只是简单包装另外一个包，那么你应该在package.json中将这个包设为同伴依赖（peerDependency）。这可以让应用开发者知道该指定哪个具体的版本。举个例子，如下所示`sass-loader`将`node-sass`指定为同伴依赖:

~~~
"peerDependencies": {
  "node-sass": "^4.0.0"
}
~~~

### Talk is cheep

以上我们已经为砍柴磨好了刀，接下来，我们动手开发一个loader。

如果我们要在项目开发中引用模板文件，那么压缩Html是十分常见的需求。分解以上需求，解析模板、压缩模板其实可以拆分给两个loader来做(单一职责)，前者较为复杂，我们就引入开源包`html-loader`，而后者，我们就拿来练手。首先我们给它取个响亮的名字--`html-minify-loader`。

接下来，按照之前介绍的步骤，首先，我们应该配置`webpack.config.js`，让webpack能识别我们的loader。当然，最最开始，我们要创建loader的文件--`src/loaders/html-minify-loader.js`。

于是，我们在配置文件中这样处理:

~~~
module: {
    rules: [{
        test: /\.html$/,
        use: ['html-loader', 'html-minify-loader'] // 处理顺序 html-minify-loader => html-loader => webpack
    }]
},
resolveLoader: {
    // 因为 html-loader 是开源 npm 包，所以这里要添加 'node_modules' 目录
    modules: [path.join(__dirname, './src/loaders'), 'node_modules']
}
~~~

接下来，我们提供html和js来测试loader:

`src/example.html`:

~~~
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    
</body>
</html>
~~~

`src/app.js`:

~~~
var html = require('./expamle.html');
console.log(html);
~~~

好了，现在我们着手处理`src/loaders/html-minify-loader.js`。前面我们说过，Loader也是一个Node模块，他导出一个函数，该函数的参数是require的源模块，处理source后把返回值交给下一个loader。所以他的模板应该是这样的:

~~~
module.exports = function (source) {
    // 处理 source ...
    return handledSource;
}
~~~

或

~~~
module.exports = function (source) {
    // 处理 source ...
    this.callback(null, handledSource)
    return handledSource;
}
~~~

> 注意：如果是处理顺序排在最后一个的 loader，那么它的返回值将最终交给 webpack 的 require，换句话说，它一定是一段可执行的 JS 脚本 （用字符串来存储），更准确来说，是一个 node 模块的 JS 脚本，我们来看下面的例子。

~~~
// 处理顺序排在最后的 loader
module.exports = function (source) {
    // 这个 loader 的功能是把源模块转化为字符串交给 require 的调用方
    return 'module.exports = ' + JSON.stringify(source);
}
~~~

整个过程相当于这个Loader把源文件

~~~
这里是 source 模块
~~~

转化为

~~~
// example.js
module.exports = '这里是 source 模块';
~~~

然后交给require调用方:

~~~
// applySomeModule.js
var source = require('example.js'); 

console.log(source); // 这里是 source 模块
~~~

而我们本次串联的两个loader钟，解析html、转化为js执行脚本的任务已经交给`html-loader`了，我们来处理html压缩问题。

作为普通node模块的Loader可以轻而易举的引用第三方库。我们使用`minimize`这个库来完成核心的压缩功能:

~~~
// src/loaders/html-minify-loader.js

var Minimize = require('minimize');

module.exports = function(source) {
    var minimize = new Minimize();
    return minimize.parse(source);
};
~~~

当然，minimize库支持一系列的压缩参数，比如comments参数指定是否需要保留注释。我们肯定不能在Loader里写死这些配置。那么`loader-utils`就该发挥作用了:

~~~
// src/loaders/html-minify-loader.js
var loaderUtils = require('loader-utils');
var Minimize = require('minimize');

module.exports = function(source) {
    var options = loaderUtils.getOptions(this) || {}; //这里拿到 webpack.config.js 的 loader 配置
    var minimize = new Minimize(options);
    return minimize.parse(source);
};
~~~

这样，我们可以在webpack.config.js中设置压缩后是否需要保留注释:

~~~
    module: {
        rules: [{
            test: /\.html$/,
            use: ['html-loader', {
                loader: 'html-minify-loader',
                options: {
                    comments: false
                }
            }] 
        }]
    },
    resolveLoader: {
        // 因为 html-loader 是开源 npm 包，所以这里要添加 'node_modules' 目录
        modules: [path.join(__dirname, './src/loaders'), 'node_modules']
    }
~~~

当然，你还可以把我们的loader写成异步的方式，这样不会阻塞其他编译进度：

~~~
var Minimize = require('minimize');
var loaderUtils = require('loader-utils');

module.exports = function(source) {
    var callback = this.async();
    if (this.cacheable) {
        this.cacheable();
    }
    var opts = loaderUtils.getOptions(this) || {};
    var minimize = new Minimize(opts);
    minimize.parse(source, callback);
};
~~~

### 总结

到这里，对于如何开发一个loader，我相信你已经有了自己的答案。总结一下，一个loader在我们项目中work需要经历以下步骤：

* 创建loader的目录及模块文件
* 在webpack中配置rule及loader的解析路径，并且要注意loader的顺序，这样在`require`指定类型文件时，我们能让处理流经过指定loader。
* 遵循原则设计和开发loader