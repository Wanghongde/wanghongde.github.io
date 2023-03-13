---
title: 手写 new
tags: JS
---

我们先写一段构造函数创建实例的代码感受下`new`

```js
function Person(uname, address) {
    this.uname = uname
    this.address = address
}

Person.prototype.sayHi = function() {
    console.log(`你好, 我是${this.uname}, 我来自${this.address}`)
}

let p = new Person('张三', '石家庄')
console.log(p.address)  // 石家庄
p.sayHi()  // 你好, 我是张三, 我来自石家庄
```

通过打印和分析，我们可以知道`new`做了这几件事

1. 创建了新对象
2. 将 `Person`的属性添加到新对象上
3. 继承`Person`原型对象上的方法
4. 返回新对象

<!--more-->

因为`new`是关键字，我们这里就模拟一个 `createFactory`方法， `Person`当成参数传递进去

```js
function createFactory(obj, ...rest) {
    // 创建一个对象，并将对象的原型对象指向 obj的原型对象
   	// 既 newObj.__proto__ = obj.prototype = Person.prototype
    let newObj = Object.create(obj.prototype)

    // 执行Person函数，将this指向 newObj 并传递参数
    obj.apply(newObj, rest)

    //	将新对象返回
    return newObj
}
```

当然这里用了一些ES6语法，我们也可以使用ES3的方式实现一下

```js
function createFactory() {
    let newObj = new Object()

    // 取出 Person函数，并赋值给fn, arguments伪数组第一项移除
    let fn = [].shift.apply(arguments)

    // 将Person的原型对象 赋值给 新对象的原型
    newObj.__proto__ = fn.prototype

    // 将Person执行，并传递参数
    fn.apply(newObj, arguments)

    return newObj
}
```

这样我们就实现了`new`的基本操作，😊

但是没完呢，我们看下如下代码：

```js
// test 1
function Person(uname, address) {
    this.uname = uname
    this.address = address

    return { age: 18 }
}
let p = new Person('张三', '石家庄')
console.log(p)  // {age: 18}

// test 2
function Person(uname, address) {
    this.uname = uname
    this.address = address

    return 'hello'
}
let p = new Person('张三', '石家庄')
console.log(p)  // {uname: '张三', address: '石家庄'}
```

这里我们看到，之前的结论其实不完美

1. 创建了新对象
2. 将 `Person`的属性添加到新对象上
3. 继承`Person`原型对象上的方法
4. 判断`Person`的**有返回值**且是**对象**，则返回对象，否则返回新对象

改造代码如下：

```js
function createFactory(obj, ...rest) {
    // 创建一个对象，并将对象的原型对象指向 obj的原型对象
   	// 既 newObj.__proto__ = obj.prototype = Person.prototype
    let newObj = Object.create(obj.prototype)

    // 执行Person函数，将this指向 newObj 并传递参数
    let result = obj.apply(newObj, rest)

    //	判断result返回值是对象则返回对象， 否则返回 newObj
    return (typeof result === 'object' && result !== null) ? result : newObj
}
```

ok，另一种也是加上判断就可以，这里我们就不写了，那就酱紫，告辞。

