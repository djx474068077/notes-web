# js执行机制

## 1. 关于JavaScript

首先了解js的几个基础概念

- JavaScript是单线程语言
- JavaScript是按照语句出现的顺序执行的
- javascript在浏览器内核中和node.js环境中执行机制不完全相同

js为什么是单线程？

&ensp;&ensp;作为浏览器脚本语言，js的主要用途就是与用户互动，以及操作DOM、BOM。这决定了它只能是单线程，否则会有很复杂的同步问题。

------------------------------------------

## 2. JavaScript事件循环

&ensp;&ensp;因为js是单线程的，意味着js的所有任务都是按顺序执行的，每次只会执行一个任务，如果当前任务执行时间过长，就会造成后面的任务阻塞。

因此，js的任务分为两类：

- 同步任务
- 异步任务

网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。 异步任务一般有以下几种：

- 最常见的有定时器函数`setTimeout`、`setInterval`和`setImmediate`

    `setTimeout`和`setInterval`都是指定在time后在任务队列里添加相关“事件”，通知主线程把相应任务放到主线程中去执行。
    `setImmediate`作为一个新的API，可以马上将相关“事件”添加到任务队列里，通知主线程把相应任务放到主线程中去执行。

- `Promise`
    
    `Promise.then`中传入了一个回调函数，将在`Promise`对象进行决议(`resolve`/`reject`)后进行异步回调。

- `MutationObserver`

    `MutationObserver`提供了监听在特定范围内DOM树发生变化事件的能力，并提供回调函数可以作出适当反应的能力。

- `process.nextTick`

    `process.nextTick`是NodeJS中的API，提供了即使执行回调的能力。

一个事件循环中的执行流程大致如下：

> 1. 同步任务和异步任务分别进入不同的执行场所，同步任务进入主线程，形成一个**执行栈**。
> 2. 主线程之外，还有一个**任务队列**（task queue），当指定的异步任务完成时，就会在**任务队列**放置一个事件。
> 3. 当**执行栈**中所有的同步任务执行完毕后，系统就会读取**任务队列**，查找里面有哪些已经满足条件的任务。那些对应的异步任务，结束等待状态，将回调函数放入**执行栈**，开始执行。
> 4. 主线程重复第3步，以上循环的过程就是js的**事件循环**（Event Loop）。

![事件循环](https://upload-images.jianshu.io/upload_images/14860853-5e9584274c060692.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp "事件循环")

----------------------------------------

## 3. JavaScript中的任务队列

通过阅读Promise/A+规范，可以得知异步的实现可分为两个机制，分别是**macro-task**(**宏任务**)和**micro-task**(**微任务**)。

Macrotasks包括: `script`（整体代码）、`setTimeout`, `setInterval`, `setImmediate`, I/O, UI Rendering；

Microtasks包括: `process.nextTick`, `Promise`, `Object.observe`, `MutationObserver`。

Macrotasks、Microtasks执行机制：

> 1. 主线程执行完后会先到**micro-task**队列中读取可执行任务
> 2. 主线程执行**micro-task**任务
> 3. 主线程到**macro-task**任务队列中读取可执行任务
> 4. 主线程执行**macro-task**任务
> 5. ...转到Step 1

这里注意的是，**UI Rendering**是在**micro-task**之后执行，需要在UI渲染之前执行的逻辑，一般采用**micro-task**异步回调方式进行调用。

同样，**micro-task**队列不宜过长，给**micro-task**队列添加过多回调阻塞**macro-task**队列的任务执行是小事，重点是这有可能会阻塞**UI Render**，导致页面不能更新。浏览器也会基于性能方面的考虑，对**micro-task**中的任务个数进行限制。

> js执行优先级：
> 1. 同步代码(按照代码顺序执行，promise构造函数立即执行，属于同步代码)
> 2. 所有的微任务（优先级`process.nextTick` > `promise.then`）
> 3. 其中的一个任务队列（根据系统性能的不同，可能会导致任务队列的优先顺序不一样，姑且与Node的事件循环一致）
> 4. 所有的微任务
> 5. 其中的一个任务队列
> 6. 所有的微任务
> 7. 其中的一个任务队列（外层任务可能已经执行完了，到了内部嵌套的其他异步任务）
> ...

![任务队列](https://user-gold-cdn.xitu.io/2017/11/21/15fdcea13361a1ec?imageslim "任务队列")


```js
    console.log('1');
    setTimeout(() => {
        console.log('2');
        Promise.resolve().then(() => {
            console.log('3');
        })
        new Promise((resolve) => {
            console.log('4');
            resolve();
        }).then(() => {
            console.log('5')
        })
    })
    Promise.resolve().then(() => {
        console.log('6');
    })
    new Promise((resolve) => {
        console.log('7');
        resolve();
    }).then(() => {
        console.log('8')
    })
    setTimeout(() => {
        console.log('9');
        Promise.resolve().then(() => {
            console.log('10');
        })
        new Promise((resolve) => {
            console.log('11');
            resolve();
        }).then(() => {
            console.log('12')
        })
    })

    // 结果：
    // 1 7 -----------宏
    // 6 8 -----------微
    // 2 4 -----------宏
    // 3 5 -----------微
    // 9 11 ----------宏
    // 10 12 ---------微
```



```js
// 例2
    async function async1() {
        console.log('async1 start');
        await async2();
        console.log('async1 end');
    }
    async function async2() {
        console.log('async2');
    }
    console.log('script start');
    setTimeout(function() {
        console.log('setTimeout');
    }, 0);
    async1();
    new Promise(function (resolve) {
        console.log('promise1');
        resolve();
    }).then(function() {
        console.log('promise2');
    })
    console.log('script end');

    // 结果：
    // script start
    // async1 start
    // async2
    // promise1
    // script end
    // async1 end
    // promise2
    // setTimeout
```


```js
// 例3（例2改动版）
    async function async1() {
        console.log('async1 start');
        await async2();
        console.log('async1 end');
    }
    async function async2() {
        console.log('async2');
        setTimeout(function() {
            console.log('async2 Timeout');
        }, 0);
    }
    console.log('script start');
    setTimeout(function() {
        console.log('setTimeout');
    }, 0);
    async1();
    new Promise(function (resolve) {
        console.log('promise1');
        setTimeout(resolve, 0);
        // resolve();
    }).then(function() {
        console.log('promise2');
    })
    console.log('script end');

    // 结果：
    // script start     主线程
    // async1 start     主线程
    // async2           主线程
    // promise1         主线程
    // script end       主线程
    // async1 end       微任务1
    // setTimeout       宏任务2
    // async2 Timeout   宏任务3
    // promise2         宏任务4
```
