# 第七章 迭代器
## 定义

## 7.2


### 7.2.1 可迭代协议
实现 Iterable 接口（可迭代协议）要求同时具备两种能力：**支持迭代的自我识别能力**和**创建实现`Iterator`接口的对象**的能力。在 `ECMAScript`中，这意味着必须暴露一个属性作为“默认迭代器”，而
且这个属性必须使用特殊的`Symbol.iterator`作为键。这个默认迭代器属性必须引用一个迭代器工厂函数，调用这个工厂函数必须返回一个新迭代器。

1) 内置类型`Iterable`接口
    > 有实现：字符串、数组、Map、Set、arguments、NodeList等DOM集合类型。

    > 未实现：Number、Object
    ```
    // 可以通过以下方式查看
        let str = 'abc'
        console.log(str[Symbol.iterator]); 
        // f values() { [native code]}
    ```

### 7.2.2 迭代器协议
迭代器是一种一次性使用的对象，用于迭代与其关联的可迭代对象。迭代器 API 使用`next()`方法在可迭代对象中遍历数据。每次成功调用 next()，都会返回一个`IteratorResult`对象，其中**包含迭代器返回的下一个值**。若不调用 next()，则无法知道迭代器的当前位置.

next() 方法返回的迭代器对象 IteratorResult 包含两个属性：`done`和`value`。
```
    // 可迭代对象
    let arr = ['foo', 'bar'];

    // 迭代器
    let iter = arr[Symbol.iterator](); 
    console.log(iter); // ArrayIterator {} 

    // 执行迭代
    console.log(iter.next()); 
    // { done: false, value: 'foo' } 
    console.log(iter.next()); 
    // { done: false, value: 'bar' } 
    console.log(iter.next()); 
    // { done: true, value: undefined }
```

每个迭代器都表示对可迭代对象的一次性有序遍历。不同迭代器的实例相互之间没有联系，只会独立地遍历可迭代对象：

```
    let arr = ['foo']; 
    let iter = arr[Symbol.iterator](); 
    console.log(iter.next()); 
    // { done: false, value: 'foo' } 
    console.log(iter.next()); 
    // { done: true, value: undefined } 
    console.log(iter.next()); 
    // { done: true, value: undefined } 
    console.log(iter.next()); 
    // { done: true, value: undefined }
```

### 7.2.3 自定义迭代器

```
class Counter{
    constructor(limit){
        this.limit = limit;
    }

    [Symbol.iterator](){
        let count = 1,limit = this.limit;
        return {
            next() {
                if(count <= limit){
                    return {done: false, value: count++};
                } else {
                    return {done: true, value: undefined};
                }
            }
        };
    }
}

let counter = new Counter(3);
for (let i of counter){console.log(3)};
// 1
// 2
// 3
```

### 7.2.4 提前终止迭代
执行迭代的结构在想让迭代器知道它不想遍历到可迭代对象耗尽时，就可以“关闭”迭代器。

## 7.3 生成器
生成器是 ECMAScript 6 新增的一个极为灵活的结构，拥有在**一个函数块内暂停和恢复代码执行的能力**。这种新能力具有深远的影响，比如，使用生成器可以自定义迭代器和实现协程。

### 7.3.1 生成器基础
生成器的形式是一个函数，函数名称前面加一个星号（*）表示它是一个生成器。
```
// 生成器函数声明
function* generatorFn() {} 
```

生成器对象实现了`Iterable`接口，它们默认的迭代是自引用的


### 7.3.2  通过`yield`中断执行
1) yield 关键字可以让生成器停止和开始执行，也是生成器最有用的地方。生成器函数在遇到 yield 关键字之前会正常执行。遇到这个关键字后，执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用 next()方法来恢复执行
2) `yield`只能在生成器内部使用
3) `yield`会接收到传给`next()`方法的第一个值,第一次调用`next`传入值不会被使用
4) 用`*`可以增强`yield`的行为，让它成为一个可迭代对象，从而一次产出一个值。
// 因为`yield*`实际上只是将一个可迭代对象序列化为一连串可以单独产出的值，所以这跟把 yield 放到一个循环里没什么不同。
5) `yield*`的值是关联迭代器返回 done: true 时的 value 属性。对于普通迭代器来说，这个值是undefined

### 7.3.3 生成器作为默认迭代器


### 7.3.4 提前终止生成器
与迭代器类似，生成器也支持“可关闭”的概念。一个实现 Iterator 接口的对象一定有 next()方法，还有一个可选的 return()方法用于提前终止迭代器。生成器对象除了有这两个方法，还有第三个方法：throw()。

### 7.4 小结




# 第十一章 期约与异步函数

## 11.1 异步编程



## 11.2 期约
2012 年 Promises/A+组织分叉（fork）了 CommonJS 的 Promises/A 建议，并以相同的名字制定了 Promises/A+规范。这个规范最终成为了ECMAScript 6 规范实现的范本。
### 11.2.1 Promises/A+ 规范
ECMAScript 6 新增的引用类型`Promise`，可以通过`new`操作符来实例化。创建新期约时需要传入`执行器（executor）`函数作为参数

### 11.2.2 期约基础



#### 4. Promise.resolve()


#### 5. Prmise.reject()

### 11.2.3 期约的实例方法
1) 在`ECMAScript`暴露的异步节奏中，任何对象都有一个then()方法。

#### 1. 实现 Thenable 接口

#### 2.Promise.prototype.then()
Promise.prototype.then()是为期约实例添加处理程序的主要方法。这个 then()方法接收最多两个参数：`onResolved`处理程序和`onRejected`处理程序。这两个参数都是可选的，如果提供的话，则会在期约分别**进入“兑现”和“拒绝”状态时执行**。

#### 3.Promise.prototype.catch()
```
let p = Promise.reject();
let onRejected = function(e) {
    setTimeout(console.log, 0, 'rejected');
}
// 这两种添加拒绝处理程序的方式是一样的
p.then(null, onRejected);
p.catch(onRejected);
```

#### 4.Promise.prototype.finally()
Promise.prototype.finally()方法用于给期约添加 onFinally 处理程序，这个处理程序在期约转换为解决或拒绝状态时都会执行。

finally 只有在传入`Promise`时会返回，其他情况按`resolve`中函数执行。

#### 5.非重入期约方法(执行顺序)

#### 6.传递解决值和拒绝理由

#### 7.拒绝期约与拒绝错误处理

#### 8.拒绝期约与拒绝错误处理

### 11.2.4 期约连锁与期约合成
#### 1. 期约连锁
把生成期约的代码提取到一个工厂函数中，就可以写成这样：
```
function delayedResolve(str) { 
 return new Promise((resolve, reject) => { 
 console.log(str); 
 setTimeout(resolve, 1000); 
 }); 
}
delayedResolve('p1 executor') 
 .then(() => delayedResolve('p2 executor')) 
 .then(() => delayedResolve('p3 executor')) 
 .then(() => delayedResolve('p4 executor')) 
// p1 executor（1 秒后）
// p2 executor（2 秒后）
// p3 executor（# 秒后）
// p4 executor（4 秒后）
```

#### 2. 期约图

#### 3. Promise.all()和 Promise.race()
##### Promise.all()
Promise.all()静态方法创建的期约会在一组期约全部解决之后再解决。这个静态方法接收一个
可迭代对象，返回一个新期约：

##### Promise.race()
Promise.race()静态方法返回一个包装期约，是一组集合中最先解决或拒绝的期约的镜像。这个
方法接收一个可迭代对象，返回一个新期约：


#### 4. 串行期约合成

### 11.2.5 期约扩展

#### 1.期约取消

#### 2. 期约进度通知


```

    class TrackablePromise extends Promise { 
        constructor(executor) { 
        const notifyHandlers = []; 
        super((resolve, reject) => { 
        return executor(resolve, reject, (status) => { 
            notifyHandlers.map((handler) => handler(status)); 
            }); 
        }); 
        this.notifyHandlers = notifyHandlers; 
        } 
        notify(notifyHandler) { 
            this.notifyHandlers.push(notifyHandler); 
            return this; 
        } 
    }

    let p = new TrackablePromise((resolve, reject, notify) => { 
        function countdown(x) { 
        if (x > 0) { 
            // notify => (status) => { notifyHandlers.map(...)};
            notify(`${20 * x}% remaining`); 
            setTimeout(() => countdown(x - 1), 1000); 
        } else { 
            resolve();
        } 
        } 

    countdown(5); 
    }); 

    // notify => notify(notifyHandler) { this.notifyHandlers.push;  ....} 
    p.notify((x) => setTimeout(console.log, 0, 'progress:', x)); 
    p.then(() => setTimeout(console.log, 0, 'completed')); 
    // （约 1 秒后）80% remaining  
    // （约 2 秒后）60% remaining 
    // （约 3 秒后）40% remaining 
    // （约 4 秒后）20% remaining 
    // （约 5 秒后）completed 

```

### 11.3.1 异步函数

#### 1.async
`async`关键字用于声明异步函数。这个关键字可以用在函数声明、函数表达式、箭头函数和方法上

1. ***异步函数如果使用 return 关键字返回了值（如果没有 return 则会返回 undefined），这个值会被 Promise.resolve()包装成一个期约对象。***
2. 异步函数的返回值期待（但实际上并不要求）一个实现 thenable 接口的对象，但常规的值也可以。


#### 2. await
因为异步函数主要针对不会马上完成的任务，所以自然需要一种暂停和恢复执行的能力。使用`await`关键字可以暂停异步函数代码的执行，等待期约解决。
await 的函数会等待执行， await 下面的函数会通过Promise包装，XXXX.then(fn())执行;


[chrome 70 和 73 async执行顺序不一致问题]：https://segmentfault.com/q/1010000016147496

1. Resolve(thenable) 不等于 Promise.resolve(thenable)
2. Resolve(non-thenable) 等价于 Promise.resolve(non-thenable)

```
70 版本：
Resolve(thenable) 转化为 Promise.resolve(thenable):

new Promise(resolve => {resolve(thenable)})

new Promise(resolve => {Promise.resolve().then(()=>{thenable.then(resolve)})})

async function async1 
    {await asyncFn()} 等价于：async1 使用 new Promise 包装

new Promise(resolve => {
    resolve(async2());
})
可以转化为：

new Promise(resolve => {Promise.resolve().then(()=>{async2.then(resolve)})})
```

```
73 版本：
 根据 TC39 最近决议，await将直接使用Promise.resolve()相同语义。
 
 async1 使用 Promise.resolve 包装：
 所以 Promise.resolve(promise) 返回 promise, 即Promise.resolve(async2()) 等价于 async2() ，所以最终得到了代码：
 async2().then(()=>{
    XXXX
 })

```

### 11.3.2 停止和恢复执行

async/await 中 await 接收函数为 thenable