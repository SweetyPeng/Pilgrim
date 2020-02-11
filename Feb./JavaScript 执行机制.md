# JavaScript执行机制
> 
## 单线程
- JavaScript是单线程语言。
- H5的Web Worker实现的“多线程”实际上指的是“多子线程”，完全受控于主线程，且不允许操作DOM。
所以javascript版的"多线程"都是用单线程模拟出来的
## Event Loop 机制
JavaScript 有一个主线程 main thread 和调用栈 call-stack(也叫执行栈)。  
所有的任务会放到调用栈中等待主线程来执行。  
1、JavaScript将任务分为同步任务和异步任务，同步任务进入主线程的执行栈中，异步任务首先到Event Table进行回调函数注册，当异步任务的触发条件满足，将回调函数从Event Table压入Event Queue中等待执行；
2、等执行栈为空以后，代表主线程执行完毕，再去Event Queue中读取第一个事件放入主线程，执行完毕再读取第二个......
因此形成JavaScript的Event Loop。
![EventLoop](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/eventloop.png?raw=true)
主线程从"任务队列"中读取事件，这个过程是循环不断的，所以整个的这种运行机制又称为Event Loop（事件循环）  
Event Loop是伴随整个源码文件生命周期的，只要当前JS在运行中，不断循环，去寻找queue里面能执行的task。
### 同步任务与异步任务
- 同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务；  
- 异步任务指的是，不进入主线程、而进入"任务队列"（task queue）的任务，只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。
### 宏任务（macro task）与微任务（micro task）
![macroTask](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/macrotask.png?raw=true)
- **常见的宏任务**： 整体的script任务、setTimeout、setInterval、setImmediate、ajax 等  
- **常见的微任务**： Promise.resolve().then()、process.nextTick、MutationObserver、Object.observe 等  
注意：在微任务中，process.nextTick的触发顺序会比其他微任务要先执行
### 谈谈setTimeout、setInterval
```javascript
setTimeout(() => {
    console.log('延时3秒');
},3000)
// console.log('执行console');
sleep(10000000)
```
- 由这段代码可知，3秒后并不会打印“延迟3秒”，因为sleep函数还在执行中；
说明：3秒后,setTimeout里的函数被会推入event queue,而event queue(事件队列)里的任务,只有在主线程空闲时才会执行。
- **setTimeout(fn,0)**
setTimeout(fn,0)的含义是，指定某个任务在主线程最早可得的空闲时间执行，意思就是不用再等多少秒了，只要主线程执行栈内的同步任务全部执行完成，栈为空就马上执行.  
注意：HTML5标准规定了setTimeout()的第二个参数的最小值（最短间隔），不得低于4毫秒，如果低于这个值，就会默认4毫秒。
- **setIntval(fn,ms)**
setIntval是循环执行的，setIntval会每隔指定的时间将注册的函数置入event queue,不是每过ms会执行一次fn,而是每过ms秒，会有fn进入event queue。
需要注意一点的是，一但setIntval的回调函数fn执行时间超过了延迟事件ms,那么就完成看不出来有时间间隔了。
### async/await
```javascript
setTimeout(function () {
  console.log('6')
}, 0)
console.log('1')
async function async1() {
  console.log('2')
  await async2()
  console.log('5')
}
async function async2() {
  console.log('3')
}
async1()
console.log('4')

// 输出结果：1-2-3-4-5-6
```
- 这是promise的语法糖，用来优化promise的回调函数
- 每个async方法都返回一个promise, await只能出现在async函数中
- async/await内部做了什么
async相当于 new Promise,await相当于then  
async函数会返回一个Promise对象，如果在函数中return一个直接量（普通变量），async会把这个直接量通过Promise.resolve()封装成Promise对象。如果你返回了promise那就以你返回的promise为准。await是在等待，等待运行的结果也就是返回值。await后面通常是一个异步操作（promise）,但是这不代表await后面只能跟异步才做，await后面实际是可以接普通函数调用或者直接量。
## 相关
### 举个例子
```javascript
new Promise(function (resolve) { 
    console.log('1')// 宏任务一
    resolve()
}).then(function () {
    console.log('3') // 宏任务一的微任务
})
setTimeout(function () { // 宏任务二
    console.log('4')
    setTimeout(function () { // 宏任务五
        console.log('7')
        new Promise(function (resolve) {
            console.log('8')
            resolve()
        }).then(function () {
            console.log('10')
            setTimeout(function () {  // 宏任务七
                console.log('12')
            })
        })
        console.log('9')
    })
})
setTimeout(function () { // 宏任务三
    console.log('5')
})
setTimeout(function () {  // 宏任务四
    console.log('6')
    setTimeout(function () { // 宏任务六
        console.log('11')
    })
})
console.log('2') // 宏任务一
```
### 参考资料
1.[这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
2.[Tasks, microtasks, queues and schedules](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)
3.[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)