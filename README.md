### Daily-FE

#### 窥探其他公司如何进行单元测试:
* ant-design
* egg

#### 以后如果觉得没什么好学的就在github里搜
awesome

#### 版本号管理
https://semver.org/lang/zh-CN/

#### 搞不懂的写法
第三课的预习中 https://zhuanlan.zhihu.com/p/26536815 以下两种写法不理解：
```
foo.call(sTarget, Array.prototype.slice.call(arguments))
```
```
Array.prototype.slice.call(document.querySelectorAll(parentSelector)).forEach(function ($p) {
    $p.addEventListener(evt, triFunction);
});
```