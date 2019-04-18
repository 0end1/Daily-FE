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

!()[https://github.com/cuantmac/Daily-FE/blob/master/img-folder/2019041801.png]

### HTTP请求模块

### 拦截器模块

## axios的设计有什么值得借鉴的地方

### 发送请求函数的处理逻辑

### Adapter的处理逻辑

## 总结