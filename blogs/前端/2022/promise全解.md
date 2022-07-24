---
title: 理解promise
date: 2020/7/20
tags:
- promise
categories:
- 前端
author: cukuyu
---


# ES6 >> Promise

## 前言

Promise是异步编程的一种解决方案，是一个对象，可以获取异步操作的消息，大大改善了异步编程的困难，避免了回调地狱，比传统的解决方案回调函数和事件更合理和更强大。

从语法上讲，Promise是一个对象，它可以获取异步操作的消息。提供了一个统一的API，各种异步操作都可以用同样的方法进行处理


**1. promise有三种状态** 

1. Pending（进行中）
2. Resolved（已完成，又称Fulfilled）
3. Rejected （已拒绝） 

**2. Promise的实例有两个过程** 

1. pending --> Resolved/Fulfiled
2. pending --> Rejected

>注意：一旦从进行中状态变为其他状态就永远不能更改状态了

## Promise的基本用法

### promise简单示例
```js
new Promise(function (resolve, reject) {
    //先要做的事情...
    if(...) resolve(result) //如果成功
    else reject(err) //如果失败
}).then(function resolve (result) {
    console.log('succses')
    //成功后要做的事情..
}).catch(function reject (err) {
    console.log('err')
    //失败后要做的事情..
}).finally(function () {
    console.log("End");
    //最终要做的事情..
});
```
> 要注意的是当一个promise对象被创建时它就开始执行方法或请求了
### promise API

#### 1. **Promise.resolve()**

Promise并非一开始就必须处于Pending状态，然后通过执行器函数才能转换为Resolved状态。通过调用Promise.resolve()静态方法，可以实例化一个解决的Promise。所以，如果希望得到一个 Promise 对象，比较方便的方法就是直接调用Promise.resolve方法。下面两个Promise实例实际上是一样的：
```js
let p1 = new Promise((resolve, reject) => resolve()); 
let p2 = Promise.resolve();
```

需要注意的是，立即resolve的 Promise 对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。

```js
setTimeout(function () {
    console.log(3);
}, 0);
Promise.resolve().then(function () {
    console.log(2);
});
console.log(1);

//输出 1 2 3
```

**Promise.resolve方法的参数分成四种情况**
1. 参数是一个 Promise 实例
如果参数是 Promise 实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例。

2. 参数是一个thenable对象   
    thenable对象指的是具有then方法的对象，比如下面这个对象
    ```js
    let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
    };
    ```
    Promise.resolve方法会将这个对象转为 Promise 对象，然后就立即执行thenable对象的then方法。
    ```js
    let thenable = {
        then: function(resolve, reject) {
            resolve(1);
        }
    };
    let p1 = Promise.resolve(thenable);
        p1.then(function(value) {
        console.log(value);  // 1
    });
    ```
3. 参数不是具有then方法的对象，或根本就不是对象     
    如果参数是一个原始值，或者是一个不具有then方法的对象，则Promise.resolve方法返回一个新的 Promise 对象，状态为resolved。
    ```js
    const p = Promise.resolve('Hello');
    p.then(function (s){
    console.log(s)
    });
    // Hello
    ```
    由于字符串Hello不属于异步操作（判断方法是字符串对象不具有 then 方法），返回 Promise 实例的状态从一生成就是resolved，所以回调函数会执行。Promise.resolve方法的参数，会同时传给回调函数

4. 不带有任何参数   
    Promise.resolve方法允许调用时不带参数，直接返回一个resolved状态的 Promise 对象。
    ```js 
    setTimeout(function () {
        console.log('three');
    }, 0);

    Promise.resolve().then(function () {
        console.log('two');
    });

    console.log('one');

    // one
    // two
    // three
    ```
    .then()函数里不返回值或者返回的不是promise，那么 then 返回的 Promise 将会成为接受状态（resolve)
    ```js
    Promise.resolve()
    .then(() => console.log(2))
    .then(() => console.log(3));
    console.log(1); // 1, 2, 3
    ```

在某些场景下，Pormise.reslove 可以极大的减少我们的心智负担。例如，将数组内的所有项都转换成 Promie
```js
[1, 2, promise1, pormise2, 3].map(item => Promise.resolve(item))
```
我们不需要判断数组内的值是否为 Promise，直接 Promise.resovle() 就是我们想要的结果。         
本质上Promise.resolve()方法就是为了更方便的创建Promise

#### 2. **Promise.reject()**
与 Promise.resolve()类似，Promise.reject()会实例化一个拒绝的Promise并抛出一个异步错误 
（这个错误不能通过 try/catch 捕获，而只能通过拒绝处理程序捕获）。下面的两个Promise实例实际上是 
一样的：
```js
let p1 = new Promise((resolve, reject) => reject()); 
let p2 = Promise.reject();
```

关键在于，Promise.reject()并没有照搬 Promise.resolve()的幂等逻辑。如果给它传一个Promise对象，则这个Promise会成为它返回的拒绝Promise的理由：
```js
setTimeout(console.log, 0, Promise.reject(Promise.resolve())); 
// Promise <rejected>: Promise <resolved>
```

#### **3. Promise.all()**
Promise.all() 方法接收一个 promise 的 iterable 类型（注：Array，Map，Set 都属于 ES6 的 iterable 类型）的输入，并且只返回一个Promise实例， 那个输入的所有 promise 的 resolve 回调的结果是一个数组。

这个Promise的 resolve 回调执行是在所有输入的 promise 的 resolve 回调都结束，或者输入的 iterable 里没有 promise 了的时候。它的 reject 回调执行是，只要任何一个输入的 promise 的 reject 回调执行或者输入不合法的 promise 就会立即抛出错误，并且 reject 的是第一个抛出的错误信息。
```js
const promise1 = Promise.resolve(3);
const promise2 = 42;
const promise3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, 'foo');
});

Promise.all([promise1, promise2, promise3]).then((values) => {
  console.log(values); // [3, 1337, "foo"]
});
```

如果参数中包含非 promise 值，这些值将被忽略，但仍然会被放在返回数组中（如果 promise 完成的话）：
```js
let p = Promise.all([1,2,3]);

let p2 = Promise.all([1,2,3, Promise.resolve(444)]);

let p3 = Promise.all([1,2,3, Promise.reject(555)]);


setTimeout(function(){
    console.log(p);
    console.log(p2);
    console.log(p3);
});
// Promise { <state>: "fulfilled", <value>: Array[3] }
// Promise { <state>: "fulfilled", <value>: Array[4] }
// Promise { <state>: "rejected", <reason>: 555 }
```

> Promise.all 的异步和同步

下面的例子中演示了 Promise.all 的异步性（如果传入的可迭代对象是空的，就是同步）：
```js
let resolvedPromisesArray = [Promise.resolve(33), Promise.resolve(44)];

let p = Promise.all(resolvedPromisesArray);
// immediately logging the value of p
console.log(p);

// using setTimeout we can execute code after the stack is empty
setTimeout(function(){
    console.log('the stack is now empty');
    console.log(p);
});

// 按顺序打印:
// Promise { <state>: "pending" }
// the stack is now empty
// Promise { <state>: "fulfilled", <value>: Array[2] }
```

如果 Promise.all 失败，也是一样的：
```js
let mixedPromisesArray = [Promise.resolve(33), Promise.reject(44)];
let p = Promise.all(mixedPromisesArray);
console.log(p);
setTimeout(function(){
    console.log('the stack is now empty');
    console.log(p);
});

// logs
// Promise { <state>: "pending" }
// the stack is now empty
// Promise { <state>: "rejected", <reason>: 44 }
```

但是，Promise.all 当且仅当传入的可迭代对象为空时为同步：
```js
var p = Promise.all([]); // will be immediately resolved
var p2 = Promise.all([1337, "hi"]); // non-promise values will be ignored, but the evaluation will be done asynchronously
console.log(p);
console.log(p2)
setTimeout(function(){
    console.log('the stack is now empty');
    console.log(p2);
});

// logs
// Promise { <state>: "fulfilled", <value>: Array[0] }
// Promise { <state>: "pending" }
// the stack is now empty
// Promise { <state>: "fulfilled", <value>: Array[2] }
```

> Promise.all() 快速返回失败行为

Promise.all 在任意一个传入的 promise 失败时返回失败。例如，如果你传入的 promise中，有四个 promise 在一定的时间之后调用成功函数，有一个立即调用失败函数，那么 Promise.all 将立即变为失败。
```js
let p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000, 'one');
});
let p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 2000, 'two');
});
let p3 = new Promise((resolve, reject) => {
  reject('reject');
});

Promise.all([p1, p2, p3]).then(values => {
  console.log(values);
}, reason => {
  console.log(reason)
});

//log:
//"reject"

//也可以用catch
Promise.all([p1, p2, p3]).then(values => {
  console.log(values);
}).catch(reason => {
  console.log(reason)
});

//log:
//"reject"
```

> 需要注意的是，forEach可以异步的执行Promise数组， 但是普通for不行
```js
let p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000, 'one');
});
let p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000, 'two');
});
let pList = [p1, p2]

// pList.forEach( async (p)=>{
//     let s = await p
//     console.log(new Date().getUTCSeconds(),s)
// })

for(const p of pList){
    let s = await p
    console.log(2,s)
}
```