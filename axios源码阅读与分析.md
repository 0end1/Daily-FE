## 概述

在前端开发过程中，我们经常会遇到需要发送异步请求的情况。而使用一个功能齐全，接口完善的HTTP请求库，能够在很大程度上减少我们的开发成本，提高我们的开发效率。

axios是一个近年来非常火的一个HTTP请求库，目前在 [GitHub](https://github.com/axios/axios)中已经拥有超过50K的star，受到了各位大佬的推荐。

今天，我们就来看下，axios是如何设计的，其中有哪些值得我们学习的地方。当前axios所有源码文件都在lib文件夹中，因此我们下文中提到的路径均是指lib文件夹中的路径。

本文的主要内容有：

* 如何使用axios
* axios的核心模块是如何设计与实现的（请求、拦截器、撤回）
* axios的设计有什么值得借鉴的地方

## 如何使用axios

想要了解axios的设计，我们首先来看下axios是如何使用的。我们通过一个简单示例来介绍下axios的API.

### 发送请求

 ~~~
 axios({
   method:'get',
   url:'http://bit.ly/2mTM3nY',
   responseType:'stream'
 })
   .then(function(response) {
   response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
 });
 ~~~
 
 这是官方的API示例。从上面的代码中我们可以看到，axios的用法与JQuery的ajax很相似(不相似的地方请看[Jquery ajax, Axios, Fetch区别之我见](https://segmentfault.com/a/1190000012836882))，都是通过返回一个Promise(也可以通过success的callback，不过建议使用Promise或者await)来继续后面的操作。
 
 这个代码示例很简单，我就不多赘述了，下面我们来看下如何添加一个过滤器函数。
 
 ### 增加拦截器(Interceptors)函数
 
 ~~~
 // 增加一个请求拦截器，注意是2个函数，一个处理成功，一个处理失败，后面会说明这种情况的原因
 axios.interceptors.request.use(function (config) {
     // 请求发送前处理
     return config;
   }, function (error) {
     // 请求错误后处理
     return Promise.reject(error);
   });
 
 // 增加一个响应拦截器
 axios.interceptors.response.use(function (response) {
     // 针对响应数据进行处理
     return response;
   }, function (error) {
     // 响应错误后处理
     return Promise.reject(error);
   });
 ~~~
 
 通过上面的示例我们可以知道：在请求发送前，我们可以针对请求的config参数进行数据处理；而在请求响应后，我们也能针对返回的数据进行特定的操作。同时，在请求失败和响应失败时，我们都可以进行特定的错误处理。
 
## axios的核心模块是如何设计与实现的

通过上面的例子，我相信大家对axios的使用方法都有了一个大致的了解。下面，我们将按照模块来对axios的设计与实现进行分析。下图是我们在这篇文章中将会涉及到的axios的文件，如果读者感兴趣的话，可以通过clone代码结合文章进行阅读，这样能够加深对相关模块的理解。

![](https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019041801.png)

### HTTP请求模块

作为核心模块，axios发送请求相关的代码位于core/dispatchRequest.js文件中。由于篇幅有限，我们选取部分重点的源码进行简单的介绍：

~~~
module.exports = function dispatchRequest(config) {
    throwIfCancellationRequested(config);

    // 其他源码

    // default adapter是一个可以判断当前环境来选择使用Node还是XHR进行请求发送的模块
    var adapter = config.adapter || defaults.adapter; 

    return adapter(config).then(function onAdapterResolution(response) {
        throwIfCancellationRequested(config);

        // 其他源码

        return response;
    }, function onAdapterRejection(reason) {
        if (!isCancel(reason)) {
            throwIfCancellationRequested(config);

            // 其他源码

            return Promise.reject(reason);
        });
};
~~~

通过上面示例代码我们可以知道，dispatchRequest方法是通过获取config.adapter来得到发送请求的模块的，我们自己也可以通过传入符合规范的adapter函数来替换掉原生的模块（虽然一般不会这么做，但也算是一个松耦合扩展点）。

在default.js中，我们能够看到相关的adapter选择逻辑，即根据当前容器中特有的一些属性和构造函数来进行判断。

~~~
function getDefaultAdapter() {
    var adapter;
    // 只有Node.js才有变量类型为process的类
    if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
        // Node.js请求模块
        adapter = require('./adapters/http');
    } else if (typeof XMLHttpRequest !== 'undefined') {
        // 浏览器请求模块
        adapter = require('./adapters/xhr');
    }
    return adapter;
}
~~~

axios中XHR模块较稳简单，为XMLHTTPRequest对象的封装，我们在这里就不再赘述了，有兴趣的同学可以自行阅读adapters/xhr.js。

### 拦截器模块

了解了dispatchRequest实现的HTTP请求发送模块，我们来看下axios是如何处理请求和响应拦截函数的。让我们看下axios中请求统一入口request函数。

~~~
Axios.prototype.request = function request(config) {

    // 其他代码

    var chain = [dispatchRequest, undefined];
    var promise = Promise.resolve(config);

    this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
        chain.unshift(interceptor.fulfilled, interceptor.rejected);
    });

    this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
        chain.push(interceptor.fulfilled, interceptor.rejected);
    });

    while (chain.length) {
        promise = promise.then(chain.shift(), chain.shift());
    }

    return promise;
};
~~~

这个函数是axios发送请求的入口，因为函数时限比较长，我就简单说一下相关的设计思路：

1. chain是一个执行队列。这个队列的初始值，是一个带有config参数的Promise.
2. 在chain执行队列中，插入了初始的发送请求的函数dispatchRequest和与之对应的undefined。后面需要增加一个undefined的原因是因为在Promise中，需要一个success和一个fail的回调函数，这个从代码 `promise = promise.then(chain.shift(), chain.shift());` 就能够看出来。因此， despatchRequest和undefined可以成为一对函数。
3. 在chain执行队列中，发送请求的函数dispatchRequest是处于中间的位置。它的前面是请求拦截器，通过unshift方法放入；它的后面是响应拦截器，通过push放入。要注意的是，这些函数是成对的放入的。

## axios的设计有什么值得借鉴的地方

### 发送请求函数的处理逻辑

在之前的章节提到过，axios在处理发送请求的dispatchRequest函数时，没有当做一个特殊的函数来对待，而是采用一视同仁的方法，将其放在队列的中间位置，从而保证了队列处理的一致性，提高了代码的可阅读性。

### Adapter的处理逻辑

在adapter的处理逻辑中，axios没有把http和xhr两个模块当成自身的模块直接在dispatchRequest中直接引用，而是通过配置的方法在default.js文件中进行默认引入。这样既保证了两个模块间的低耦合性，同时又能够为今后用户自定义请求发送模块保留了余地。

## 总结

本文对axios相关的使用方式，设计思路和实现方式进行了详细的介绍。读者能够通过上述文章，了解axios的设计思想，同时能够在axios的代码中，学习到关于模块封装和交互等相关的经验。