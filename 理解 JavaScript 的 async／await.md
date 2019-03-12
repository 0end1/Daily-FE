## 理解 JavaScript 的 async/await

### async和await在干什么

任意一个名称都是有意义的，从字面意思来理解，async是“异步”的简写，而await是async wait的简写。所以应该很好理解async用于声明一个异步的的func，而await用于等待一个异步执行完成。

另外还有一个很有意思的语法规定，await只能出现在async函数中。然后细心的朋友会发现一个问题，如果await只能出现在async函数中，那这个async函数应该怎么调用？

如果需要一个await来调用，那这个调用的外面必须得再包一个async函数，然后进入死循环，永无出头之日。。。

如果async不需要await来调用，那async到底起啥作用呢？

#### async起什么作用

这个问题的关键在于，async函数是怎么处理它的返回值的。

我们当然希望它能直接通过return语句返回我们想要的值，但如果真的是这样，那就没await什么事了。所以，写段代码试试，看看它到底返回什么：

~~~
async function testAsync() {
    return "hello async";
}

const result = testAsync();
console.log(result);
~~~

看到输出就恍然大悟了——输出的是一个Promise对象

~~~
c:\var\test> node --harmony_async_await .
Promise { 'hello async' }
~~~

所以async函数返回的是一个Promise对象。如果在函数中return一个直接量，async会把这个直接量通过promise.resolve()封装成Promise对象。

async函数返回的是一个Promise对象，所以在最外层不能用await获取其返回值的情况下，我们当然应该用原来的方式；then()链来处理这个Promise对象，就像这样：

~~~
testAsync().then(v => {
    console.log(v);    // 输出 hello async
});
~~~

想一下，如果async函数没有返回值，又该如何？很容易想到，他会返回Promise.resolve(undefined)。

联想一下Promise的特点——无等待，所以在没有await的情况下执行async函数，他会立即执行，返回一个Promise对象，并且，绝不会阻塞后面语句的执行。这和普通的Promise对象并无二致。

那么下一个关键点就在于await关键字了。

#### await到底在等啥

一般来说await是在等待一个async函数完成，不过按照[语法说明](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await)，await等待的是一个表达式，这个表达式的计算结果是Promise对象或者其他值（也就是没有特殊规定）。

因为async函数返回一个Promise对象，所以await可以用于等待一个async函数的返回值——这也可以说是在等async函数，但要清楚，他等的实际是一个返回值。注意到await不仅仅用于等Promise对象，还可以等任意表达式的结果，所以，await后面实际是可以接普通函数调用或者直接量的。所以下面这个示例完全可以正确运行：

~~~
function getSomething() {
    return "something";
}

async function testAsync() {
    return Promise.resolve("hello async");
}

async function test() {
    const v1 = await getSomething();
    const v2 = await testAsync();
    console.log(v1, v2);
}

test();
~~~

#### await 等到了要等的，然后呢

await等到了他要等的东西，一个Promise对象，或者其他值，然后呢？我们不得不说，await是个运算符，用于组成表达式，await表达式的运算结果取决于他等的东西。

如果他等到的不是一个Promise对象，那await表达式的运算结果就是他等到的东西。

如果他等到的是一个Promise对象，await就忙起来了，他会阻塞后面的代码，等着Promise对象resolve，然后得到resolve的值，作为await表达式的运算结果。

> 看到上面阻塞一词，心慌了吧。。。放心，这就是awiat必须用在async函数中的原因。async函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个Promise对象中异步执行。

### async/await 帮我们干了啥

#### 做个简单的比较

上面已经说明了async会将其后的函数的返回值封装成一个Promise对象，而await会等待这个Promise完成，并将其resolve的结果返回出来。

现在举例，用setTimeout模拟耗时的异步操作，先来看看不用async/await该如何写：

~~~
function takeLongTime() {
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);
    });
}

takeLongTime().then(v => {
    console.log("got", v);
});
~~~

如果改成async/await呢，会是这样：

~~~
function takeLongTime() {
    return new Promise(resolve => {
        setTimeout(() => resolve("long_time_value"), 1000);
    });
}

async function test() {
    const v = await takeLongTime();
    console.log(v);
}

test();
~~~

眼尖的同学已经发现 takeLongTime()没有申明为async。实际上，takeLongTime()本身就是返回Promise对象，加不加async结果都一样，如果没明白，请回过头再去看看上面的"async起什么作用"。

又一个疑问产生了，这两段代码，两种方式对异步调用的处理（实际上就是对Promise对象的处理）差别并不明显，甚至使用 async/await 还要多写一些代码，那他的优势在哪？

#### async/await 的优势在于处理 then 链

单一的Promise链并不能展现 async/await 的优势，但是，如果需要处理由多个Promise组成的then链的时候，优势就能体现出来了（很有意思，Promise通过then链来解决多层回调问题，现在又用 async/await 来进一步优化它）。

假设一个业务，分多个步骤完成，每个步骤都是异步的，而且依赖上一个步骤的结果。我们仍然用 setTimeout 来模拟异步操作：

~~~
/**
 * 传入参数 n，表示这个函数执行的时间（毫秒）
 * 执行的结果是 n + 200，这个值将用于下一步骤
 */
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
~~~

现在用Promise方式来实现这三个步骤的处理：

~~~
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();

// c:\var\test>node --harmony_async_await .
// step1 with 300
// step2 with 500
// step3 with 700
// result is 900
// doIt: 1507.251ms
~~~

如果用 async/await 来实现呢，会是这样：

~~~
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();
~~~

结果和之前的Promise实现是一样的，但是这个代码看起来清晰很多，几乎跟同步代码一样。

#### 还有更酷的

现在把业务需求改一下，仍然是三个步骤，但每一个步骤都需要之前每个步骤的结果。

~~~
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m + n);
}

function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k + m + n);
}
~~~

这回先用 async/await 来写：

~~~
async function doIt() {
    console.log("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}
~~~

来对比一下Promise方式实现会是什么样子：

~~~
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        })
}
~~~

有没有感觉有点复杂？那一堆参数的处理就是Promise方案的死穴!