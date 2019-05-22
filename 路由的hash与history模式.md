## 前端路由的实现

### 前言

前端路由是现代SPA应用的必备功能，每个前端框架都有相应的实现，例如vue-router，react-router。

我们不想探究vue-router或者react-router的实现，因为不管哪种实现无外乎用兼容性更好的hash实现或者是H5 Histrory实现，框架只需要做相应的封装就可以。

```
提前声明：我们没有对传入的参数进行及时判断而规避错误，也没有考虑兼容性问题，仅仅是对核心方法进行了实现
```

### 1.hash路由

hash路由一个明显的标志就是带有"#"标志，我们主要通过监听url地址栏中hash值的变化来进行路由的跳转。(DT:根据url的变化亦可用来监测PV)

hash的优势是兼容性更好，在老版的IE中都可以运行，缺点是不够美观，带有"#"，而且hash路由更像是hack而非标准，相信随着发展，更加标准化的history api会逐步蚕食掉hash路由的市场。

#### 1.1初始化Class

我们先来初始化一个路由：
```
class Routers {
    constructor() {
        // 以键值对形式存储路由
        this.routes = {};
        // 当前路由的URL
        this.currentUrl = '';
    }
}
```

#### 1.2实现路由hash存储与执行

在初始化完毕后我们需要思考两个问题：

* 1.将路由的hash以及相应的callback函数存储
* 2.触发路由hash变化后，执行相应的callback函数
```
class Routers {
    constructor() {
        this.routes = {};
        this.currentUrl = '';     
    }
    
     // 将Path路径与相应的callback函数存储
     route(path, callback) {
        this.routes[path] = callback || function() {};
     }
     
     // 刷新
     refresh() {
        // 获取当前URL地址栏中的hash路径
        this.currentUrl = location.hash.slice(1) || '/';
        // 执行当前hash路径的callback函数
        this.routes[this.currentUrl]();
     }
}
```

#### 1.3监听对应事件

我们只需要在实例化Class的时候监听上面的事件即可
```
class Routers {
    constructor() {
        this.routes = {};
        this.currentUrl = '';
        this.refresh = this.refresh.bind(this);
        window.addEventListener('load',this.refresh,false);
        window.addEventListener('hashchange',this.refresh,false);
    }
    
     route(path, callback) {
        this.routes[path] = callback || function() {};
     }
     
     refresh() {
        this.currentUrl = location.hash.slice(1) || '/';
        this.routes[this.currentUrl]();
     }
}
```
完整示例：
点击这里[hash router](https://codepen.io/xiaomuzhu/pen/KorqGx/) on CodePen

### 2.增加回退功能

上一节我们只实现了路由的简单功能，没有常用的回退与前进功能，所以我们来对其进行继续改造。

#### 2.1实现后退功能

我们需要创建一个数组history来存储过往的hash路由,例如/blue，并且创建一个指针currentIndex来根据*后退*和*前进*功能移动来指向不同的hash路由。
```
class Routers {
    constructor() {
        this.routes = {};
        this.currentUrl = '';
        // 记录出现过的hash
        this.history = [];
        // 作为指针，默认指向history的末尾,根据后退前进指向History中不同的hash
        this.currentIndex = this.history.length - 1; 
        this.refresh = this.refresh.bind(this);
        window.addEventListener('load',this.refresh,false);
        window.addEventListener('hashchange',this.refresh,false);
    }
    
     route(path, callback) {
        this.routes[path] = callback || function() {};
     }
     
     refresh() {
        this.currentUrl = location.hash.slice(1) || '/';
        // 将当前Hash值推入数组存储
        this.histroy.push(this.currentUrl);
        // 指针向前移动
        this.currentIndex++; 
        this.routes[this.currentUrl]();
     }
     
     // 后退功能
     backOff() {
        // 指针不可小于0
        if(this.currentIndex < 0) {
            this.currentIndex = 0;
        } else {
            this.currentIndex--;
        }
        // 随着后退，hash值也相应随之变化
        location.hash = `#${this.history[this.currentIndex]}`;
        this.routes[this.history[this.currentIndex]]();
     }
}
```

看起来我们实现的不错，可是出现了bug，再后退的时候我们需要点击两下。点击查看bug示例[hash router](https://codepen.io/xiaomuzhu/pen/mxQBod/)。

bug的原因在于我们每次后退都会执行相应的callback函数，这会触发refresh()执行，因此每当我们后退，history中都会被push进新的路由Hash,currentIndex也会向前移动，这显然不是我们想要的。

我们必须做一个判断，如果时候退的话，我们就只执行回调函数，不需要添加数组和移动指针：
```
class Routers {
    constructor() {
        this.routes = {};
        this.currentUrl = '';
        this.history = [];
        this.currentIndex = this.history.length - 1; 
        this.refresh = this.refresh.bind(this);
        // 默认不是后退操作
        this.isBack = false;
        window.addEventListener('load',this.refresh,false);
        window.addEventListener('hashchange',this.refresh,false);
    }
    
     route(path, callback) {
        this.routes[path] = callback || function() {};
     }
     
     refresh() {
        this.currentUrl = location.hash.slice(1) || '/';
        if(!this.isBack) {
            // 如果不是后退操作,且当前指针小于数组总长度,直接截取指针之前的部分储存下来
            // 此操作来避免当点击后退按钮之后,再进行正常跳转,指针会停留在原地,而数组添加新hash路由
            // 避免再次造成指针的不匹配,我们直接截取指针之前的数组
            // 此操作同时与浏览器自带后退功能的行为保持一致
            if (this.currentIndex < this.history.length - 1)
               this.history = this.history.slice(0, this.currentIndex + 1);
            this.histroy.push(this.currentUrl);
            this.currentIndex++; 
        }
        this.routes[this.currentUrl]();
        this.isBack = false;
     }
     
     backOff() {
        // 后退操作设置为true
        this.isBack = true;
        if(this.currentIndex < 0) {
            this.currentIndex = 0;
        } else {
            this.currentIndex--;
        }
        location.hash = `#${this.history[this.currentIndex]}`;
        this.routes[this.history[this.currentIndex]]();
     }
}
```
查看完整示例[hash router](https://codepen.io/xiaomuzhu/pen/VXVrxa/) on CodePen。

前进的部分就不实现了，思路已经比较清晰。可以看出,hash路有这种方式确实有点繁琐，所以H5提供了History API供我们使用。

### 3.H5新路由方案

#### 3.1History API

我们可以直接在浏览器里查询出History API的方法和属性。

<img width="227" alt="wechat screenshot_20181202111941" src="https://user-images.githubusercontent.com/21993931/49335220-621b1e00-f624-11e8-8b1f-007c4a90b926.png">

想要全面了解可以去MDN查询[History API资料](https://developer.mozilla.org/zh-CN/docs/Web/API/History)。

我们只简单看下常用的API

```
window.history.back();
window.history.forward();
window.history.go(-3);
```
history.pushState用于在浏览历史记录里添加记录，但并不触发跳转，此方法接收三个参数：
```
state:一个与指定网址相关的对象（我理解为可以随便自定义），popstate事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可传null
title:新页面的标题，但是目前所有浏览器都忽略这个值，因此这里可填写null
url:新的网址，必须与当前网页处于同一个域内。浏览器地址栏将显示该地址。
```

history.replaceState方法的参数与pushState方法一模一样，区别是它是用来修改浏览历史中当前记录，而非添加记录，同样不触发跳转。

popstate事件，每当同一个文档的浏览历史(即history对象)出现变化时，就会触发popstate事件。需要注意的是，仅仅调用pushState或replaceState方法是不会触发该事件的。只有用户点击浏览器的前级或者回退按钮，或者使用js调用back,forward,go方法时才会触发。另外，该事件只针对同一个文档，如果浏览历史的切换导致加载不同的文档，该事件也不会触发

以上API介绍摘自[history对象](https://javascript.ruanyifeng.com/bom/history.html#toc0)。

#### 3.2新标准下路由的实现

mini一个history api的路由：
```
class Routers {
    constructor() {
        this.routes = {};
        this._bindPopState();
    }
    
    init(path) {
        history.replaceState({path: path}, null, path);
        this.routes[path] && this.routes[path]();
    }

    route(path, callback) {
        this.routes[path] = callback || function() {};
    }

    go(path) {
        history.pushState({path: path}, null, path);
        this.routes[path] && this.routes[path]();
    }

    _bindPopState() {
        window.addEventListener('popstate', e => {
            const path = e.state && e.state.path;
            this.routes[path] && this.routes[path]();
        });
    }
}
```
点击查看H5路由[H5 Router](https://codepen.io/xiaomuzhu/pen/QmJorQ/)on CodePen。

## 小结

我们大致探究了前端路由的两种实现方式，在没有兼容性要求的情况下，显然history api实现的路由是个较好的选择。

想更深入的了解前端路由可以阅读[vue-router代码](https://github.com/vuejs/vue-router/blob/dev/src/index.js)。