## bind

一句话介绍bind:

> bind()方法会创建一个新函数。当这个新函数被调用时，bind()的第一个参数将作为它运行时的this,之后的的一系列参数将会在传递的实参前传入作为它的参数。（来自于MDN）

由此我们可以首先得出bind函数的两个特点：

1. 返回一个函数
2. 可以传入参数

### 返回函数的模拟实现

从第一个特点开始，我们举个例子：

~~~
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

// 返回了一个函数
var bindFoo = bar.bind(foo); 

bindFoo(); // 1
~~~

我们来写第一版的代码：

~~~
// 第一版
Function.prototype.bind2 = function (context) {
    var self = this;
    return function () {
        return self.apply(context);
    }
}
~~~

之所以 ` return self.apply(context) ` ,是考虑到绑定函数可能是有返回值的，依然是这个例子：

~~~
var foo = {
    value: 1
};

function bar() {
	return this.value;
}

var bindFoo = bar.bind(foo);

console.log(bindFoo()); // 1
~~~

### 传参的模拟实现

这个就让人有点费解了，我在bind的时候，是否可以传参呢？我在执行bind返回的函数的时候，可不可以传参呢？让我们先看一个例子：

~~~
var foo = {
    value: 1
};

function bar(name, age) {
    console.log(this.value);
    console.log(name);
    console.log(age);

}

var bindFoo = bar.bind(foo, 'daisy');
bindFoo('18');
// 1
// daisy
// 18
~~~

函数需要传name和age两个参数，竟然还可以在bind的时候，只传一个name，在执行返回的函数的时候，再传另一个参数age!

这可咋办？不急，我们用arguments进行处理：

~~~
// 第二版
Function.prototype.bind2 = function (context) {

    var self = this;
    // 获取bind2函数从第二个参数到最后一个参数
    var args = Array.prototype.slice.call(arguments, 1);

    return function () {
        // 这个时候的arguments是指bind返回的函数传入的参数
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(context, args.concat(bindArgs));
    }
}
~~~

### 构造函数效果的模拟实现

完成了这两点，最难的部分到啦！因为bind还有一个特点，就是

> 一个绑定函数也能使用new操作符创建对象: 这种行为就像把原函数当成构造器。提供的this值被忽略，同时调用时的参数被提供给模拟函数。

也就是说当bind返回的函数作为构造函数的时候，bind指定的this值会失效，但传入的参数依然生效。举个例子：

~~~
var value = 2;

var foo = {
    value: 1
};

function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}

bar.prototype.friend = 'kevin';

var bindFoo = bar.bind(foo, 'daisy');

var obj = new bindFoo('18');
// undefined
// daisy
// 18
console.log(obj.habit);
console.log(obj.friend);
// shopping
// kevin
~~~

注意：尽管在全局和foo中都声明了value值，最后依然返回了undefind，说明绑定的this失效了，如果大家了解new的模拟实现，就会知道这个时候的this已经指向obj。

所以我们可以通过修改返回的函数的原型来实现，让我们写一下：

~~~
// 第三版
Function.prototype.bind2 = function (context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
        // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
        // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    fBound.prototype = this.prototype;
    return fBound;
}
~~~

如果对原型链有困惑，可以查看[《JavaScript深入之从原型到原型链》](https://github.com/mqyqingfeng/Blog/issues/2)

### 构造函数效果的优化实现

但是在这个写法中，我们直接将fBound.prototype = this.prototype, 我们直接修改fBound.prototype的时候，也会直接修改绑定函数的prototype。这个时候，我们可以通过一个空函数来进行中转：

~~~
// 第四版
Function.prototype.bind2 = function (context) {

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
~~~

到此为止，大问题都已经决绝了，给自己一个赞！o(￣▽￣)ｄ

### 两个小问题

1. 调用 bind的不是函数咋办？

不行，我们要报错！

~~~
if (typeof this !== "function") {
  throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
}
~~~

2. 要在线上用

别忘了做个兼容：

~~~
Function.prototype.bind = Function.prototype.bind || function () {
    ……
};
~~~

### 最终代码

~~~
Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
~~~