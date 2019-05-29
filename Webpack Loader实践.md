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

不同的模块会有不同的依赖形式。比如

### Talk is cheep

### 总结