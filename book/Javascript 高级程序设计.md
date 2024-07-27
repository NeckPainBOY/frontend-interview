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

#### 1.Promise.prototype.then()
Promise.prototype.then()是为期约实例添加处理程序的主要方法。这个 then()方法接收最多两个参数：`onResolved`处理程序和`onRejected`处理程序。
