## 移动端h5之rem布局/ px2rem

### Rem布局之媒体匹配

最早的时候用rem适配方式，通过手动设置媒体查询对不同设备进行font-size的设置：
// 自适应
// ------------------------
```
html{
  font-size: 38px;
}
@media only screen and (min-width: 320px) {
  html {
    font-size: 42.666px !important;
  }
}
@media only screen and (min-width: 360px) {
  html {
    font-size: 48px !important;
  }
}
@media only screen and (min-width: 375px) {
  html {
    font-size: 50px !important;
  }
}
@media only screen and (min-width: 414px) {
  html {
    font-size: 55.2px !important;
  }
}
@media only screen and (min-width: 480px) {
  html {
    font-size: 64px !important;
  }
}
@media only screen and (min-width: 560px) {
  html {
    font-size: 74.666px !important;
  }
}
@media only screen and (min-width: 640px) {
  html {
    font-size: 85.333px !important;
  }
}
@media only screen and (min-width: 720px) {
  html {
    font-size: 96px !important;
  }
}
@media only screen and (min-width: 750px) {
  html {
    font-size: 100px !important;
  }
}
@media only screen and (min-width: 800px) {
  html {
    font-size: 106.666px !important;
  }
}
@media only screen and (min-width: 960px) {
  html {
    font-size: 128px !important;
  }
}
```
这是很古老的方法，可以看出我们的设计稿是750px的，以750为基准设置font-size为100。
我们可以看到这个方式是分段式的，并不连续，所以想过难说理想。

### rem布局之自动设置fontsize

所以有了下面这段根据屏幕尺寸自动设置fontsize的方法，直接贴代码：
```
(function (doc, win) {
        var docEl = doc.documentElement,
            resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
            recalc = function () {
                var clientWidth = docEl.clientWidth;
                if (!clientWidth) return;
                if(clientWidth>=640){
                    docEl.style.fontSize = '100px';
                }else{
                    docEl.style.fontSize = 100 * (clientWidth / 640) + 'px';
                }
            };

        if (!doc.addEventListener) return;
        win.addEventListener(resizeEvt, recalc, false);
        doc.addEventListener('DOMContentLoaded', recalc, false);
    })(document, window);
```
上面这段代码的意思就是，当页面的宽度大于640px时，页面中html的fontsize都被设置为100px，否则，会被设置为100 * (当前页面宽度 / 640)；

### rem布局之开阔视野

该方案的使用相当简单，把下面这段已压缩过的原生JS(仅1kb)放到HTML的head标签中即可(注：不要手动设置viewport,该方案自动帮你设置)；

<script>!function(e){function t(a){if(i[a])return i[a].exports;var n=i[a]={exports:{},id:a,loaded:!1};return e[a].call(n.exports,n,n.exports,t),n.loaded=!0,n.exports}var i={};return t.m=e,t.c=i,t.p="",t(0)}([function(e,t){"use strict";Object.defineProperty(t,"__esModule",{value:!0});var i=window;t["default"]=i.flex=function(e,t){var a=e||100,n=t||1,r=i.document,o=navigator.userAgent,d=o.match(/Android[\S\s]+AppleWebkit\/(\d{3})/i),l=o.match(/U3\/((\d+|\.){5,})/i),c=l&&parseInt(l[1].split(".").join(""),10)>=80,p=navigator.appVersion.match(/(iphone|ipad|ipod)/gi),s=i.devicePixelRatio||1;p||d&&d[1]>534||c||(s=1);var u=1/s,m=r.querySelector('meta[name="viewport"]');m||(m=r.createElement("meta"),m.setAttribute("name","viewport"),r.head.appendChild(m)),m.setAttribute("content","width=device-width,user-scalable=no,initial-scale="+u+",maximum-scale="+u+",minimum-scale="+u),r.documentElement.style.fontSize=a/2*s*n+"px"},e.exports=t["default"]}]);  flex(100, 1);</script>

这是阿里团队的高清方案布局代码，所谓高清方案就是根据屏幕DPR(设备像素比，比如DPR=2时，1个CSS像素由2个物理像素点组成)，动态设置html的font-size，同时根据设备DPR调整页面缩放值，进而达到高清效果。

### rem布局之px2rem

我们在写rem布局的时候，尽管我们设置了基准的font-size，但是我们还是要去换算一下。有没有办法直接自动转换成rem呢？

有的：

px2rem的原理：

![原理](../原理.png "原理")

使用方式(待验证)：

npm install px2rem-loader --save-dev

// utils.js

```
const px2remLoader = {
    loader: 'px2rem-loader',
    option: {
        remUnit: 37.5,
        dpr: 1
    }
}
```
loader的加载方式请参考官方文档。