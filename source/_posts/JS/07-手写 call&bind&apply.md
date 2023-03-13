---
title: 手写 call&bind&apply
tags: JS
---

在 JS 中提到修改 `this`指向，首先想到的就是 `bind`、`call`、`apply`。

面试也经常会问到三个的区别，那我们就一起来边使用边手动书写一下吧。

```js
let obj = {
	address: '石家庄'
}

var address = '北京'

let fn = function(num1, num2) {
	console.log(this.address, num1, num2)
}

fn(1, 2) // 北京 1 2
fn.call(obj, 11, 22) // 石家庄 11 22
```

通过上边案例我们可以发现call做了这几件事

1. 修改了 `fn` 的 `this`指向
2. 传递了参数
3. 调用了`fn`函数

<!--more-->

那我们也试着在`Function`的原型对象上挂载一个`myCall`

```js
Function.prototype.myCall = function(context){
    // context 是 传递的参数 这里指向 obj
    // this 是函数的调用者 这里指向 fn
    // 所以这句话的意思是 给 obj 增加一个属性fn,并赋值成 fn函数
    context.fn = this  
    // 因为外部相当于调用了fn，所以这里我们也调用一下
    const result = context.fn()
    // context.fn属性只是暂用一下，应该删除掉
    delete context.fn
    
    return result
}
```

我们实现了函数调用，并且调用者指向了 `context`也就是`obj`了

```js
let obj = {
    address: '石家庄'
}

var address = '北京'

let fn = function() {
    console.log(this.address)
}

Function.prototype.myCall = function(context){
    // context 是 传递的参数 这里指向 obj
    // this 是函数的调用者 这里指向 fn
    // 所以这句话的意思是 给 obj 增加一个属性fn,并赋值成 fn函数
    context.fn = this  
    // 因为外部相当于调用了fn，所以这里我们也调用一下
    const result = context.fn()
    // context.fn属性只是暂用一下，应该删除掉
    delete context.fn
    
    return result
}

fn.myCall(obj) // 石家庄
```

这里需要注意 call传递参数时传递多个，而apply是传递一个数组

```js
Function.prototype.myCall = function(context){
    // context 是 传递的参数 这里指向 obj
    // this 是函数的调用者 这里指向 fn
    // 所以这句话的意思是 给 obj 增加一个属性fn,并赋值成 fn函数
    context.fn = this  
    // 准备用户传递过来的参数
    let arr = []
    for(let i = 1; i < arguments.length; i++) {
        arr.push(arguments[i])
    }
    // 在调用函数前 准备出传递的参数
    // 因为外部相当于调用了fn，所以这里我们也调用一下
    context.fn(...arr)
    // context.fn属性只是暂用一下，应该删除掉
    delete context.fn
}
```

那apply我们只需要稍微改造就可以了

```js
Function.prototype.myApply = function(context, args = []){
    // context 是 传递的参数 这里指向 obj
    // this 是函数的调用者 这里指向 fn
    // 所以这句话的意思是 给 obj 增加一个属性fn,并赋值成 fn函数
    context.fn = this  

    // 因为外部相当于调用了fn，所以这里我们也调用一下
    context.fn(...args)
    // context.fn属性只是暂用一下，应该删除掉
    delete context.fn
}
```

最后我们来看一下`bind`

```js
let obj = {
    address: '石家庄'
}

let fn = function(one, two, three) {
    console.log(this.address, one, two, three)
}

let fn2 = fn.bind(obj, 1, 2, 3)

fn2()
```

- 修改了 `this`指向 
- 返回的是一个函数
- 传递多个参数

所以 `bind`的返回值一定是一个函数，并且函数内，需要执行`fn`函数

```js
Function.prototype.myBind = function(context) {
    let fn = this
    
    let arr = []
    for(let i = 1; i < arguments.length; i++) {
        arr.push(arguments[i])
    }
    
    return function() {
        return fn.apply(context, arr)
    }
}
```

当然我们这里只是实现了基本的应用，当需要真实修改`this`指向时还是要使用原生JS的语法