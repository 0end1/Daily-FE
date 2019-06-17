### 前端学习锦囊
* 红宝书，犀牛书多看几遍并应用
* 写博客，给别人讲清楚
* 分析一个成熟的库(例如Jquery，Vue等)的实现，并写成博客
* 攻克知识难点，写系列学习笔记
* 算法/数据结构的书看到二叉树（前三五章）
* 以后如果觉得没什么好学的就在github里搜 awesome
* github上star了一个 JavaScript深入系列
* 《你不知道的JavaScript》
* 曾探《JvaScript设计模式与开发实践》（已购）
* 刷题素材：https://juejin.im/post/5cb87f9df265da03555c78ec
* 拆解 JavaScript 中的异步模式：https://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651232427&idx=1&sn=84691cca63745d36ac97b959a6fd696c&chksm=bd49412f8a3ec8398eaf2cf7eb43d5ab7ac8a98528d93a4e4a9a81f4763449c656533b0df46b&scene=0&xtrack=1&key=729816feb00e845f263602d67c5b1e8fc53208b485837e617d58d9c945e4e08286b8a15c592f4a85980d466c046f78d56740f9924014a884830b970d14cd2f7ebce48ec78d793f9f57ef8a5dbc72b505&ascene=1&uin=MTgyNDg2NTU0Mg%3D%3D&devicetype=Windows+10&version=62060825&lang=en&pass_ticket=q9QFAxZZRcNfQ8BTDptfRQXqj4Ad3N9YH3S8h0RhVQitTBmzg2Peizqeorw%2FzwVI
* 我认为比较好的讲解Webpack的文章(基于V3)：https://segmentfault.com/a/1190000012081469


### 知识点收集

* http1.1与http1.0的区别，https与http的区别 https://www.cnblogs.com/heluan/p/8620312.html

* 标准浏览器的事件模型，事件如何绑定 https://segmentfault.com/a/1190000005654451

* 变量提升的原理

* 富文本编辑器如何解决 XSS 问题 (设置标签白名单) https://jsxss.com/en/index.html

* IndexedDB 使用与出坑指南 https://juejin.im/post/5a9d65916fb9a028e46e257a

* 九种跨域解决方式：https://segmentfault.com/a/1190000011145364#articleHeader0
  1. JOSNP
  2. CORS
  3. iframe（细分三种）
  4. nginx（细分两种）
  5. postMessage
  6. websocket协议跨域
 
* OOCSS的概念 https://www.w3cplus.com/css/oocss-concept

* BEM命名规范 https://juejin.im/entry/58e605d80ce46300584a1afb

* 版本号管理 https://semver.org/lang/zh-CN/

* 让你对webpck和tapable理解提升一个层次的文章 https://juejin.im/post/5beb8875e51d455e5c4dd83f?from=groupmessage

* call、apply、bind使用和区别 https://juejin.im/post/5a9640335188257a7924d5ef

* 如何理解 JavaScript 中的 this 关键字？ https://www.zhihu.com/question/19636194 (加深认知:[JavaScript的this原理](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html))

* 前后端分离容易产生的问题及前后端心态总结 https://www.cnblogs.com/hetaojs/p/10616773.html

* Key值在列表渲染中的作用：https://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651231974&idx=1&sn=ec6a9af65dc1804f281a4e7d1bde945a&chksm=bd494f628a3ec6749add615ff10a3e28ce39eb778b64120a3deb45166c91b71220582c1f82de&scene=0&xtrack=1&key=81c8654a52f48fc2a54c11772533098db17ddac6a10fb410930c49385893d3da1e1c16abbc4106c8933a956125d47f93b1081487dce3eda463ffb0916c4b6c1e13a0317116926a5e3bb59fd0cd6108f5&ascene=1&uin=MTgyNDg2NTU0Mg%3D%3D&devicetype=Windows+10&version=62060739&lang=en&pass_ticket=5bkc3i68mKG9%2FcoVpkvguzJz8o6wz9m9YF94tDA6Hcf9vcZYLDC%2BZVYmha9FhPqJ
    * 不加Key值，在数组的末尾追加元素，之前已经渲染的元素不会重新渲染。但如果在头部或者中间插入元素，整个List被删除重新渲染。
    * 添加Key值，在数组末尾、头部或者中间位置插入元素，其他已经渲染的元素都不会被重新渲染。
    
* 为什么要对图片进行base64编码：https://www.zhangxinxu.com/wordpress/2012/04/base64-url-image-%E5%9B%BE%E7%89%87-%E9%A1%B5%E9%9D%A2%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96/   

* 单例模式：https://segmentfault.com/a/1190000012842251

* 此处的0，在Node中约为1ms，在浏览器中约为4ms

~~~
setTimeout(() => {},0)
~~~

* Vuex、Flux、Redux、Redux-saga、Dva、MobX https://juejin.im/post/5c18de8ef265da616413f332#heading-23(看的心累)

* vue组件间通信六种方式（完整版）

* 重绘与重排 https://juejin.im/post/5c15f797f265da61141c7f86#heading-12

* GZIP原理 https://juejin.im/post/5b793126f265da43351d5125#heading-3

* 一些比较基础的JS基础测试题目 https://github.com/lydiahallie/javascript-questions/blob/master/README-zh_CN.md