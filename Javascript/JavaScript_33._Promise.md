# JavaScript 33. Promise


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d3f2ece8cb46244f114f0ffaac2a2ca3.png)


##  1. 前言
Promise是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了Promise对象。 所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。本文主要介绍JavaScript(JS)中Promise，以及相关的使用方法，以及示例代码。

##  2. 简介
Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。

Promise对象有以下两个特点：

 1. 对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：`Pending`（进行中）、`Resolved`（已完成，又称Fulfilled）和`Rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
 2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：**从Pending变为Resolved和从Pending变为Rejected**。只要这两种情况发生，状态就固定，会一直保持这个结果。就算再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同。

**注意**：事件的特点是，如果错过了它，再去监听，是得不到结果的。 有了Promise对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。此外，Promise对象提供统一的接口，使得控制异步操作更加容易。 Promise也有一些缺点。首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于Pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）。 如果某些事件不断地反复发生，一般来说，使用stream模式是比部署Promise更好的选择。

##  3. 浏览器支持
由于 Promise 是 ES6 新增加的，所以一些旧的浏览器并不支持，苹果的 Safari 10 和 Windows 的 Edge 14 版本以上浏览器才开始支持 ES6 特性。

以下是 Promise 浏览器支持的情况：
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/dbacafbdd350a02e28446b0d15706808.png)

##  4. 构造 Promise
现在我们新建一个 Promise 对象：

```bash
new Promise(function (resolve, reject) {
    // 要做的事情...
});
```
通过新建一个 Promise 对象好像并没有看出它怎样 "更加优雅地书写复杂的异步任务"。我们之前遇到的异步任务都是一次异步，如果需要多次调用异步函数呢？例如，如果我想分三次输出字符串，第一次间隔 1 秒，第二次间隔 4 秒，第三次间隔 3 秒：

```bash
setTimeout(function () {
    console.log("First");
    setTimeout(function () {
        console.log("Second");
        setTimeout(function () {
            console.log("Third");
        }, 3000);
    }, 4000);
}, 1000);
```
这段程序实现了这个功能，但是它是用 "**函数瀑布**" 来实现的。可想而知，在一个复杂的程序当中，用 "函数瀑布" 实现的程序无论是维护还是异常处理都是一件特别繁琐的事情，而且会让缩进格式变得非常冗赘。

现在我们用 Promise 来实现同样的功能：

```bash
new Promise(function (resolve, reject) {
    setTimeout(function () {
        console.log("First");
        resolve();
    }, 1000);
}).then(function () {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            console.log("Second");
            resolve();
        }, 4000);
    });
}).then(function () {
    setTimeout(function () {
        console.log("Third");
    }, 3000);
});
```

## 5. Promise 的使用
Promise 构造函数只有一个参数，是一个函数，这个函数在构造之后会直接被异步运行，所以我们称之为起始函数。起始函数包含两个参数 `resolve` 和 `reject`。

```bash
new Promise(function (resolve, reject) {
    var x = 0;
    var y = 1;
    if (y == 0) reject("除数不能为零");
    else resolve(x / y);
}).then(function (value) {
    console.log("x / y = " + value);
}).catch(function (err) {
    console.log(err);
}).finally(function () {
    console.log("End");
});
```
输出结果：

```bash
x / y = 0
End
```
`resolve` 和 `reject` 都是函数，其中调用 resolve 代表一切正常，reject 是出现异常时所调用的。Promise 类有 `.then()`、.`catch()` 和 `.finally()` 三个方法，这三个方法的参数都是一个函数，.then() 可以将参数中的函数添加到当前 Promise 的正常执行队列，.catch() 则是设定 Promise 的异常处理序列，.finally() 是在 Promise 执行的最后一定会执行的序列。 .then() 传入的函数会按顺序依次执行，有任何异常都会直接跳到 catch 序列。

实例1

```bash
```bash
// 第一步：model层的接口封装
const promise = new Promise((resolve, reject) => {
    // 这里做异步任务（比如ajax 请求接口。这里暂时用定时器代替）
    setTimeout(function() {
        var data = { retCode: 0, msg: 'qianguyihao' }; // 接口返回的数据
        if (data.retCode == 0) {
            // 接口请求成功时调用
            resolve(data);
        } else {
            // 接口请求失败时调用
            reject({ retCode: -1, msg: 'network error' });
        }
    }, 100);
});
// 第二步：业务层的接口调用。这里的 data 就是 从 resolve 和 reject 传过来的，也就是从接口拿到的数据
promise.then(data => {
    // 从 resolve 获取正常结果
    console.log(data);
}).catch(data => {
    // 从 reject 获取异常结果
    console.log(data);
});




```



实例2
```bash
new Promise(function (resolve, reject) {
    console.log(1111);
    resolve(2222);
}).then(function (value) {
    console.log(value);
    return 3333;
}).then(function (value) {
    console.log(value);
    throw "An error";
}).catch(function (err) {
    console.log(err);
});
```
执行结果：

```bash
1111
2222
3333
An erro
```
`resolve()` 中可以放置一个参数用于向下一个 then 传递一个值，then 中的函数也可以返回一个值传递给 then。但是，如果 then 中返回的是一个 Promise 对象，那么下一个 then 将相当于对这个返回的 Promise 进行操作，这一点从刚才的计时器的例子中可以看出来。

`reject()` 参数中一般会传递一个异常给之后的 catch 函数用于处理异常。

但是请注意以下两点：

 - resolve 和 reject 的作用域只有起始函数，不包括 then 以及其他序列；
 - resolve 和 reject 并不能够使起始函数停止运行，别忘了 return。

## 6. Promise 函数
上述的 "计时器" 程序看上去比函数瀑布还要长，所以我们可以将它的核心部分写成一个 Promise 函数：

```bash
function print(delay, message) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            console.log(message);
            resolve();
        }, delay);
    });
}
```

然后我们就可以放心大胆的实现程序功能了：

```bash
print(1000, "First").then(function () {
    return print(4000, "Second");
}).then(function () {
    print(3000, "Third");
});
```
## 7. 异步函数
异步函数（async function）是 ECMAScript 2017 (ECMA-262) 标准的规范，几乎被所有浏览器所支持，除了 Internet Explorer。

在 Promise 中我们编写过一个 Promise 函数：

```bash
function print(delay, message) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            console.log(message);
            resolve();
        }, delay);
    });
}
```
然后用不同的时间间隔输出了三行文本：

```bash
print(1000, "First").then(function () {
    return print(4000, "Second");
}).then(function () {
    print(3000, "Third");
});
```
我们可以将这段代码变得更好看：

```bash
async function asyncFunc() {
    await print(1000, "First");
    await print(4000, "Second");
    await print(3000, "Third");
}
asyncFunc();
```
哈！这岂不是将异步操作变得像同步操作一样容易了吗！

这次的回答是肯定的，异步函数 `async function` 中可以使用 `await` 指令，await 指令后必须跟着一个 Promise，异步函数会在这个 Promise 运行中暂停，直到其运行结束再继续运行。

异步函数实际上原理与 Promise 原生 API 的机制是一模一样的，只不过更便于程序员阅读。

处理异常的机制将用 `try-catch` 块实现：

```bash
async function asyncFunc() {
    try {
        await new Promise(function (resolve, reject) {
            throw "Some error"; // 或者 reject("Some error")
        });
    } catch (err) {
        console.log(err);
        // 会输出 Some error
    }
}
asyncFunc();
```
如果 Promise 有一个正常的返回值，await 语句也会返回它：

```bash
async function asyncFunc() {
    let value = await new Promise(
        function (resolve, reject) {
            resolve("Return value");
        }
    );
    console.log(value);
}
asyncFunc();
```
程序会输出:

```bash
Return value
```

## 8. 实战
### 8.1 Promise 处理 多次 Ajax 请求

```bash
/*
  基于Promise发送Ajax请求
*/
function queryData(url) {
    var promise = new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return;
            if (xhr.readyState == 4 && xhr.status == 200) {
                // 处理正常情况
                resolve(xhr.responseText); // xhr.responseText 是从接口拿到的数据
            } else {
                // 处理异常情况
                reject('接口请求失败');
            }
        };
        xhr.responseType = 'json'; // 设置返回的数据类型
        xhr.open('get', url);
        xhr.send(null); // 请求接口
    });
    return promise;
}
// 发送多个ajax请求并且保证顺序
queryData('http://localhost:3000/api1')
    .then(
        data1 => {
            console.log(JSON.stringify(data1));
            // 请求完接口1后，继续请求接口2
            return queryData('http://localhost:3000/api2');
        },
        error1 => {
            console.log(error1);
        }
    )
    .then(
        data2 => {
            console.log(JSON.stringify(data2));
            // 请求完接口2后，继续请求接口3
            return queryData('http://localhost:3000/api3');
        },
        error2 => {
            console.log(error2);
        }
    )
    .then(
        data3 => {
            // 获取接口3返回的数据
            console.log(JSON.stringify(data3));
        },
        error3 => {
            console.log(error3);
        }
    );
```
### 8.2 函数返回Promise对象
返回值为一个 Promise 对象的函数称作 Promise 函数，它常常用于开发基于异步操作的库。

```bash
/*
  基于Promise发送Ajax请求
*/
function queryData(url) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return;
            if (xhr.readyState == 4 && xhr.status == 200) {
                // 处理正常情况
                resolve(xhr.responseText);
            } else {
                // 处理异常情况
                reject('接口请求失败');
            }
        };
        xhr.responseType = 'json'; // 设置返回的数据类型
        xhr.open('get', url);
        xhr.send(null); // 请求接口
    });
}
// 发送多个ajax请求并且保证顺序
queryData('http://localhost:3000/api1')
    .then(
        data1 => {
            console.log(JSON.stringify(data1));
            return queryData('http://localhost:3000/api2');
        },
        error1 => {
            console.log(error1);
        }
    )
    .then(
        data2 => {
            console.log(JSON.stringify(data2));
            // 注意：返回的是 Promise 实例对象，
            // 如果返回的是具体值时，则会产生一个新的默认的 promise实例，来调用这里的then，
            // 确保可以继续进行链式操作获取结果。相当于return 'cjavapy';
            return new Promise((resolve, reject) => {
                resolve('cjavapy');
            });
        },
        error2 => {
            console.log(error2);
        }
    )
    .then(data3 => {
        console.log(data3);
    });
```

### 8.3 Promise方法(Promise.all()和Promise.race())

 - `Promise.all()`：并发处理多个异步任务，所有任务都执行成功，才能得到结果。
 - `Promise.race()`: 并发处理多个异步任务，只要有一个任务执行成功，就能得到结果。

Promise.all()示例：

```bash
var p1 = Promise.resolve(1),
    p2 = Promise.resolve(2),
    p3 = Promise.resolve(3);
Promise.all([p1, p2, p3]).then(function (results) {
    console.log(results);  // [1, 2, 3]
});
```
者

```bash
/*
  封装 Promise 接口调用
*/
function queryData(url) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return;
            if (xhr.readyState == 4 && xhr.status == 200) {
                // 处理正常结果
                resolve(xhr.responseText);
            } else {
                // 处理异常结果
                reject('服务器错误');
            }
        };
        xhr.open('get', url);
        xhr.send(null);
    });
}
var promise1 = queryData('http://localhost:3000/p1');
var promise2 = queryData('http://localhost:3000/p2');
var promise3 = queryData('http://localhost:3000/p3');
Promise.all([promise1, promise2, promise3]).then(result => {
    console.log(result);
});
```
Promise.race()示例：

```bash
var p1 = Promise.resolve(1),
    p2 = Promise.resolve("error1"),
    p3 = Promise.reject("error2");
Promise.race([p1, p2, p3]).then(function (results) {
    console.log(results);  //1
});
```
或者

```bash
/*
  封装 Promise 接口调用
*/
function queryData(url) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() {
            if (xhr.readyState != 4) return;
            if (xhr.readyState == 4 && xhr.status == 200) {
                // 处理正常结果
                resolve(xhr.responseText);
            } else {
                // 处理异常结果
                reject('服务器错误');
            }
        };
        xhr.open('get', url);
        xhr.send(null);
    });
}
var promise1 = queryData('http://localhost:3000/p1');
var promise2 = queryData('http://localhost:3000/p2');
var promise3 = queryData('http://localhost:3000/p3');
Promise.race([promise1, promise2, promise3]).then(result => {
    console.log(result);
});
```

参考：

 - [JavaScript(JS) Promise介绍及使用](https://www.cjavapy.com/article/2234/)
 - [JavaScript Promise](https://www.runoob.com/js/js-promise.html)

