---
title: javascript EventLoop
---
---
<font face="行楷"></font>
* js是单线程语言
* js的执行机制 EventLoop

### 为什么是单线程?
---
Js主要用途是交互，以及操作DOM，这决定了其只能是单线程的，否则会产生复杂的同步问题。 例：假设js同时有两个线程，一个线程在DOM添加内容，另一个删除了该节点，则问题来了，浏览器以谁为准？
为利用多核cpu计算能力，HTML5提出Web Worker标准，允许js脚本创建多个线程，但子线程完全受主线程控制，且不得操作DOM，所以 并没有违反js单线程本质。
### 任务队列
---
js将所有任务分两种 同步任务（sync）和异步任务（async）。同步至在主线程上排队执行的任务，只有当一个任务执行完毕，才能执行后一个任务；异步指的是 不进入主线程，而是进入任务队列（task queue）,只有任务队列通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。
异步执行允许机制：
1.所有同步任务在主线程上执行，形成一个执行栈（excution context stack）。
2.主线程之外，存在一个‘任务队列’，只要异步任务有了允许结果，就在该任务队列放置一个事件。
3.一旦执行栈中所有的同步任务执行完毕，系统就会读取任务队列，看看有哪些事件。那些对应的异步任务，于是结束等待，按顺序进入执行栈，开始执行。
4.主线程不断重复上面三步。
### 事件和回调函数
---
"任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件。

"任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取。

所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

"任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上是自动的，只要执行栈一清空，"任务队列"上第一位的事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程。

### Event Loop
---
主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）。
![博客41.JPG](http://upload-images.jianshu.io/upload_images/6396529-43b43ee235285c5c.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图中，主线程允许时，产生堆（heap）和栈（stack）。调用栈遇到DOM操作，ajax请求以及setTimeout等WebAPIS的时候就会交给浏览器内核其他模块处理，webkit内核在js执行引擎之外，还有一个重要的模块webcore模块。对于图中WebAPIs提到的三种API，webcore分别提供了DOM Binding、network、timer模块来处理底层实现。等到这些模块处理完这些操作的时候讲回调函数放入队列当中，之后等栈中的task执行完毕后再其执行任务队列中的回调函数。
### 从setTimeout看事件循环机制
---
![博客42.JPG](http://upload-images.jianshu.io/upload_images/6396529-a50fd4503521f1fa.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先main()函数的执行上下文入栈。
![博客43.png](http://upload-images.jianshu.io/upload_images/6396529-24c8aefb4ccce521.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码接着执行，遇到console.log(‘Hi’),此时log(‘Hi’)入栈，console.log方法只是一个webkit内核支持的普通的方法，所以log(‘Hi’)方法立即被执行。此时输出’Hi’。

![博客44.png](http://upload-images.jianshu.io/upload_images/6396529-fd9074190ba32826.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当遇到setTimeout的时候，执行引擎将其添加到栈中。
![博客45.JPG](http://upload-images.jianshu.io/upload_images/6396529-32866bf837001001.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调用栈发现setTimeout是之前提到的WebAPIs中的API，因此将其出栈之后将延时执行的函数交给浏览器的timer模块进行处理。
![博客46.png](http://upload-images.jianshu.io/upload_images/6396529-ae6f94864defd0cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

timer模块去处理延时执行的函数，此时执行引擎接着执行将log(‘SJS’)添加到栈中，此时输出’SJS’。
![博客47.png](http://upload-images.jianshu.io/upload_images/6396529-9331e72e75425752.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当timer模块中延时方法规定的时间到了之后就将其放入到任务队列之中，此时调用栈中的task已经全部执行完毕。
![博客48.png](http://upload-images.jianshu.io/upload_images/6396529-66a1451b02e2d10d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![博客49.png](http://upload-images.jianshu.io/upload_images/6396529-8cafd2a74292019d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

调用栈中的task执行完毕之后，执行引擎会接着看执行任务队列中是否有需要执行的回调函数。这里的cb函数被执行引擎添加到调用栈中，接着执行里面的代码，输出’there’。等到执行结束之后再出栈。

>小结：
* 所有的代码都要通过函数调用栈中调用执行。
* 当遇到前文中提到的APIs的时候，会交给浏览器内核的其他模块进行处理。
* 任务队列中存放的是回调函数。
* 等到调用栈中的task执行完之后再回去执行任务队列之中的task

[*面试题*](https://juejin.im/post/58cf180b0ce4630057d6727c "摘录自 80%应聘者都不及格的面试题")
```javascript
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
      console.log(new Date, i);
    }, 1000);
}
console.log(new Date, i);
```
>分析：
* i=0时，满足条件，执行循环体里面的代码，发现setTimeout，将其出栈之后把延时执行的函数交给Timer模块进行处理。
* 当i=1,2,3,4时均满足条件，和i=0相同，因此timer模块里有5个相同的延时执行的函数。
* 当i=5时，不满足条件，因此for循环结束,console.log(new Date, i)入栈，此时的i已经变成了5。因此输出5。
* 此时1s已经过去，timer模块将5个回调函数按照注册的顺序返回给任务队列。
* 执行引擎去执行任务队列中的函数，5个function依次入栈执行之后再出栈，此时的i已经变成了5。因此几乎同时输出5个5
* 因此等待的1s的时间其实只有输出第一个5之后需要等待1s，这1s的时间是timer模块需要等到的规定的1s时间之后才将回调函数交给任务队列。等执行栈执行完毕之后再去执行任务对列中的5个回调函数。这期间是不需要等待1s的。因此输出的状态就是：5 -> 5,5,5,5,5，即第1个 5 直接输出，1s之后，输出 5个5；
### 宏任务（macro-task）和微任务（micro-task）
---
```javascript
(function test() {
    setTimeout(function() {console.log(4)}, 0);
    new Promise(function executor(resolve) {
        console.log(1);
        for( var i=0 ; i<10000 ; i++ ) {
            i == 9999 && resolve();
        }
        console.log(2);
    }).then(function() {
        console.log(5);
    });
    console.log(3);
})()
```
上述代码中，setTimeout和Promise都称之为任务源，来自不同任务源的回调函数会被放进不同的任务队列里面。
注意：setTimeout的回调函数被放进setTimeout的任务队列里面。尔对于Promise,他的回调函数并不是传进去的executer函数，而是其异步执行的then方法里面的参数，被放进Promise的任务队列之中。也就是说Promise的第一个参数并不会放进Promise的任务队列之中，而是就在当前队列之中执行。

1.macro-task包括：script（整体代码），setTimeout，setInterval，setlmmediate,I/O，UIrendering.
2.micro-task包括：process.nectTick,Promise,Object.obserbver,MutationObserver

事件循环的顺序是从script开始第一次循环，随后全局上下文进入函数调用栈，碰到macro-task就将其交给处理它的模块处理完之后将回调函数放进macro-task的队列之中。碰到micro-task也是将其回调函数放进micro-task的队列之中。直到函数调用栈清空只剩全局执行上下文，然后开始执行所有的micro-task。当所有可执行的micro-task执行完毕之后。循环再次执行macro-task中的一个任务队列，执行完之后再执行所有的micro-task，就这样一直循环。
按照这种分类方式：JS的执行机制是
* 执行一个宏任务，过程中如果遇到微任务，就将其放到微任务的【事件队列】里。
* 当前宏任务执行完后，会查看文任务的【事件队列】，并将里面全部的微任务一次执行完
* 重复以上步骤
  ![bV1TKz.png](http://upload-images.jianshu.io/upload_images/6396529-0766e64fb4603188.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![1460000010913954.png](http://upload-images.jianshu.io/upload_images/6396529-f0bb21b0058863fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
分析代码执行步骤：
1.script任务源执行，全局上下文入栈
2.script任务源代码执行时遇到setTimeout，作为一个macro-task，将其回调函数放入自己的队列中。
3.接着又script在执行时又遇到Promise实例。*Promise构造函数中的第一个参数是在当前任务直接执行，不会放入队列之中，此时输出1.
4.在for循环里遇到fresolve函数，函数入栈后出栈，此时Promise状态变成Fufilled.代码接着执行遇到cosole.log(2),输出2。
5.接着执行，代码遇到then方法，其回调函数作为micro-task入栈，进入Promise的任务队列之中。
6.代码接着执行，此时遇到console.log(3),输出3。
7.输出3之后第一个宏任务script的代码执行完毕，这时候开始开始执行所有在队列之中的微任务。then的回调函数入栈执行完毕之后出栈，这时候输出5。
8.这时候所有的micro-task执行完毕，第一轮循环结束。第二轮循环从setTimeout的任务队列开始，setTimeout的回调函数入栈执行完毕之后出栈，此时输出4。
###  js定时器
---
1.setTimeout(fn, x)表示延迟x毫秒之后执行fn。*在指定时间内, 将任务放入事件队列,等待js引擎空闲后被执行*
```javascript
(function testSetTimeout() {
    const label = 'setTimeout';
    console.time(label);
    setTimeout(() => {
        console.timeEnd(label);
    }, 10);
    for(let i = 0; i < 100000000; i++) {}
})();
```
HTML5规范规定最小延迟时间不能小于4ms，即x如果小于4，会被当做4来处理。 不过不同浏览器的实现不一样，比如，Chrome可以设置1ms，IE11/Edge是4ms。

setTimeout注册的函数fn会交给浏览器的定时器模块来管理，延迟时间到了就将fn加入主进程执行队列，如果队列前面还有没有执行完的代码，则又需要花一点时间等待才能执行到fn，所以实际的延迟时间会比设置的长。如在fn之前正好有一个超级大循环，那延迟时间就不是一丁点了
2.setInterval的实现机制跟setTimeout类似，只不过setInterval是重复执行的.
```javascript
(function testSetInterval() {
    let i = 0;
    const start = Date.now();
    const timer = setInterval(() => {
        i += 1;
        i === 5 && clearInterval(timer);
        console.log(`第${i}次开始`, Date.now() - start);
        for(let i = 0; i < 100000000; i++) {}
        console.log(`第${i}次结束`, Date.now() - start);
    }, 100);
})();
```
```javascript
第1次开始 100
第1次结束 1089
第2次开始 1091
第2次结束 1396
第3次开始 1396
第3次结束 1701
第4次开始 1701
第4次结束 2004
第5次开始 2004
第5次结束 2307
```
对于setInterval(fn, 100)容易产生一个误区：并不是上一次fn执行完了之后再过100ms才开始执行下一次fn。 事实上，setInterval并不管上一次fn的执行结果，而是每隔100ms就将fn放入主线程队列，而两次fn之间具体间隔多久就不一定了，跟setTimeout实际延迟时间类似，和JS执行情况有关。
虽然每次fn执行时间都很长，但下一次并不是等上一次执行完了再过100ms才开始执行的，实际上早就已经等在队列里了。
另外可以看出，*当setInterval的回调函数执行时间超过了延迟时间，已经完全看不出有时间间隔了*。
如果setTimeout和setInterval都在延迟100ms之后执行，那么谁先注册谁就先执行回调函数
注意：多个定时器如不及时清除（clearTimeout），会存在干扰，使延迟时间更加捉摸不透。所以，不管定时器有没有执行完，及时清除已经不需要的定时器是个好习惯。

3.setImmediate setImmediate（目前IE11/Edge支持、Nodejs支持，Chrome不支持，其他浏览器未测试）延迟可以在1ms以内，而setTimeout有最低4ms的延迟，所以setImmediate比setTimeout(0)更早执行回调函数。不过在Nodejs中，两者谁先执行都有可能，原因是Nodejs的事件循环和浏览器的略有差异。
```javascript
(function testSetImmediate() {
    const label = 'setImmediate';
    console.time(label);
 
    setImmediate(() => {
        console.timeEnd(label);
    });
})();
```
>注意*setImmediate指定的任务总是在下一次Event Loop时执行*

### 常用异步模型
---
*requestAnimationFrame*
requestAnimationFrame并不是定时器，但和setTimeout很相似，在没有requestAnimationFrame的浏览器一般都是用setTimeout模拟。
requestAnimationFrame跟屏幕刷新同步，大多数屏幕的刷新频率都是60Hz，对应的requestAnimationFrame大概每隔16.7ms触发一次，如果屏幕刷新频率更高，requestAnimationFrame也会更快触发。基于这点，在支持requestAnimationFrame的浏览器还使用setTimeout做动画显然是不明智的。
*Promise*
Promise是很常用的一种异步模型，如果我们想让代码在下一个事件循环执行，可以选择使用setTimeout(0)、setImmediate、requestAnimationFrame(Chrome)和Promise。

而且Promise的延迟比setImmediate更低，意味着Promise比setImmediate先执行.
可以肯定的是，在各JS环境中，Promise都是最先执行的，setTimeout(0)、setImmediate和requestAnimationFrame顺序不确定
*process.nextTick*
process.nextTick是不会进入异步队列的，而是直接在主线程队列尾强插一个任务，虽然不会阻塞主线程，但是会阻塞异步任务的执行，如果有嵌套的process.nextTick，那异步任务就永远没机会被执行到了。

// 在NODE下执行以下代码
```javascript
console.log('golb1');

setImmediate(function() {
    console.log('immediate1');
    process.nextTick(function() {
        console.log('immediate1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('immediate1_promise');
        resolve();
    }).then(function() {
        console.log('immediate1_then')
    })
})

setTimeout(function() {
    console.log('timeout1');
    process.nextTick(function() {
        console.log('timeout1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('timeout1_promise');
        resolve();
    }).then(function() {
        console.log('timeout1_then')
    })

    setTimeout(function() {
    	console.log('timeout1_timeout1');
    process.nextTick(function() {
        console.log('timeout1_timeout1_nextTick');
    })
    setImmediate(function() {
    	console.log('timeout1_setImmediate1');
    })
    });
})

new Promise(function(resolve) {
    console.log('glob1_promise');
    resolve();
}).then(function() {
    console.log('glob1_then')
})

process.nextTick(function() {
    console.log('glob1_nextTick');
})
```
//输出
```javascript
golb1
glob1_promise
glob1_nextTick
glob1_then
timeout1
timeout1_promise
timeout1_nextTick
timeout1_then
immediate1
immediate1_promise
immediate1_nextTick
immediate1_then
timeout1_timeout1
timeout1_timeout1_next
timeout1_setImmediate1
```
*参考*
[【转向Javascript系列】从setTimeout说事件循环模型](http://www.alloyteam.com/2015/10/turning-to-javascript-series-from-settimeout-said-the-event-loop-model/)
[ 【JavaScript 运行机制详解】：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
[【javascript定时器与执行机制详解】](http://www.alloyteam.com/2016/05/javascript-timer/)
[【# js事件循环机制】](https://segmentfault.com/a/1190000012399886)
[【深入理解JS引擎的执行机制】](https://segmentfault.com/a/1190000012806637#articleHeader4)
[【# js 执行机制 事件循环】](https://segmentfault.com/a/1190000012676859)



