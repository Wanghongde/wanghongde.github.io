---
title: 手写Promise
tags: JS
---

### Promise 的基本语法

promise 是 es6 提供的解决异步编程的新语法， 主要解决两个问题，第一个解决之前回调地狱(层层嵌套)， 第二个不需要立即指定回调函数，还可以在异步操作执行完提供。

promise 可以有三个状态 `pendding`、`resolved`、`rejected`，并且是不可逆的。

1. 由`pendding` 变成 `resolved`: 

   a. 构造函数内调用 resolve

2. 由 `pendding` 变成 `rejected`

​       a. 构造函数调用reject

​       b. 构造函数抛出错误

<!-- more -->

### Promise 构造函数的书写

之前说了 promise是有状态的`pendding`，并且需要返回数据暂定`null`

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态  
    this.initValue()
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null
  }
}
```

promise的初始状态是`pendding`，并且 `resolve`和`reject`函数的this需要指向promise实例，防止随着函数执行环境而改变

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态
    this.initValue()
    // 修改resolve和reject的this
    this.initBind()

    executor(this.resolve, this.reject)
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null
  }

  initBind() {
    this.resolve = this.resolve.bind(this)
    this.reject = this.reject.bind(this)
  }
}
```

当执行resolve则状态改为`resolved`并返回结果，当执行reject状态改为`rejected`也返回结果

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态
    this.initValue()
    // 修改resolve和reject的this
    this.initBind()

    executor(this.resolve, this.reject)
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null
  }

  initBind() {
    this.resolve = this.resolve.bind(this)
    this.reject = this.reject.bind(this)
  }

  resolve(value) {
    this.PromiseState = 'resolved'
    this.PromiseResult = value
  }

  reject(reason) {
    this.PromiseState = 'rejected'
    this.PromiseResult = reason
  }
}
```

简单测试

```js
let p = new MyPromise((resolve, reject) => {
  resolve('成功')
})
console.log(p)  // MyPromise { PromiseState: 'resolved', PromiseResult: 111}

let p2 = new MyPromise((resolve, reject) => {
  reject('失败')
})
console.log(p2)  // MyPromise { PromiseState: 'rejected', PromiseResult: '失败'}
```

但是有个问题，如果测试代码改成这样， promise的状态由`pendding`改成了`resolved`， 接着又新改成了`rejected`，这跟我们的初衷违背`promise的状态不可逆`

```js
let p = new MyPromise((resolve, reject) => {
  resolve('成功')
  reject('失败')
})
console.log(p)  // MyPromise { PromiseState: 'rejected', PromiseResult: '失败'}
```

既然发现了问题，那么解决其实很简单，只需要判断修改前的状态是否是`pendding` 就可以， 如果是 `pendding` 则状态可以修改，如果不是那么状态定死不可修改

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态
    this.initValue()
    // 修改resolve和reject的this
    this.initBind()

    executor(this.resolve, this.reject)
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null
  }

  initBind() {
    this.resolve = this.resolve.bind(this)
    this.reject = this.reject.bind(this)
  }

  resolve(value) {
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'resolved'
    this.PromiseResult = value
  }

  reject(reason) {
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'rejected'
    this.PromiseResult = reason
  }
}

let p = new MyPromise((resolve, reject) => {
  resolve('成功11')
  reject('失败11')
})
console.log(p)  // MyPromise { PromiseState: 'resolved', PromiseResult: '成功11'}
```

之前说过，promise执行`reject`有两种情况，`1. 构造函数调用` 、`2. 抛出异常`

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态
    this.initValue()
    // 修改resolve和reject的this
    this.initBind()

    // 构造函数内抛出异常，直接执行 catch
    try {
      executor(this.resolve, this.reject)
    } catch {
      this.reject()
    }
    
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null
  }

  initBind() {
    this.resolve = this.resolve.bind(this)
    this.reject = this.reject.bind(this)
  }

  resolve(value) {
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'resolved'
    this.PromiseResult = value
  }

  reject(reason) {
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'rejected'
    this.PromiseResult = reason
  }
}

let p = new MyPromise((resolve, reject) => {
  throw '直接报错'
})
console.log(p)  // MyPromise {PromiseState: 'rejected', PromiseResult: '直接报错' }
```

ok，我们基本搞定了Promise类的功能，待会我们再完善。

### Promise.then() 的实现

promise 的then可以接受两个回调函数，一个是成功回调，一个是失败回调。

1. 当promise的状态为 `resolved` 则执行**成功**回调函数，当promise的状态为 `rejected` 则执行**失败** 回调函数

   ```js
// then 接受两个回调函数 onResolve成功  onRejct失败
MyPromise.prototype.then = function(onResolve, onReject) {
    // then接收两个函数，所以需要校验下
    // 1. 传递的是函数 则正常使用
    // 2. 传递的不是函数 则修改成函数
    onResolve = typeof(onResolve) === 'function' ? onResolve : val => val
    onReject = typeof(onReject) === 'function' ? onReject : reason => {throw reason}
     if(this.PromiseState === 'resolved') {
       onResolve(this.PromiseResult)
     } else if(this.PromiseState === 'rejected') {
       onReject(this.PromiseResult)
     }
      console.log('1. 当构造函数里有定时器时，先执行了 then 函数')
   }
   
   
   new MyPromise((resolve, reject) => {
     resolve('成功')
   }).then(res => {
     console.log(res) // 成功
   })
   
   new MyPromise((resolve, reject) => {
     reject('错了')
   }).then(() => {}, err => {
     console.log(err)  // 错了
   })
   
   new MyPromise((resolve, reject) => {
     throw '错了22'
   }).then(() => {}, err => {
     console.log(err)  // 错了22
   })
   
   // ******** 有个问题 ********
   new MyPromise((resolve, reject) => {
     setTimeout(() => {
       console.log('2. 再执行的异步定时器')
       resolve('成功')
     }, 1000)
   
   }).then(res => {
     console.log(res)
   })
   // 打印
   // 先执行: 1. 当构造函数里有定时器时，先执行了 then 函数
   // 等一秒钟后: 2. 再执行的异步定时器
   ```
2. 如果resolve或者reject在定时器里，则等定时器结束在执行

   当promise为`pendding`，则说明定时器调用，并将定时器内的方法挂载起来

   当定时器时间结束，则promise的状态发生改变，重新调用定时器里的方法

   由于then可以串联，所以可能挂载的方法很多，最好用数组进行存储

```js
class MyPromise {
  constructor(executor) {
    // 初始化promise的数据和状态
    this.initValue()
    // 修改resolve和reject的this
    this.initBind()

    // 构造函数内抛出异常，直接执行 catch
    try {
      executor(this.resolve, this.reject)
    } catch(err) {
      this.reject(err)
    }
    
  }

  initValue() {
    this.PromiseState = 'pendding'
    this.PromiseResult = null

    // 创建两个容器，存放函数
    this.onResolveArr = []
    this.onRejectArr = []
  }

  initBind() {
    this.resolve = this.resolve.bind(this)
    this.reject = this.reject.bind(this)
  }

  resolve(value) {
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'resolved'
    this.PromiseResult = value

    // 状态发生改变说明定时器到时间了, 容器里有函数, 调用函数
    if(this.onResolveArr.length) {
      this.onResolveArr.shift()(this.PromiseResult)
    }
  }

  reject(reason) {
    console.log('88')
    if(this.PromiseState !== 'pendding') {
      return 
    }
    this.PromiseState = 'rejected'
    this.PromiseResult = reason

    // 状态发生改变说明定时器到时间了, 容器里有函数, 调用函数
    if(this.onRejectArr.length) {
      this.onRejectArr.shift()(this.PromiseResult)
    }
  }
}

MyPromise.prototype.then = function(onResolve, onReject) {
  // then接收两个函数，所以需要校验下
  // 1. 传递的是函数 则正常使用
  // 2. 传递的不是函数 则修改成函数
  onResolve = typeof(onResolve) === 'function' ? onResolve : val => val
  onReject = typeof(onReject) === 'function' ? onReject : reason => {throw reason}

  if(this.PromiseState === 'resolved') {
    onResolve(this.PromiseResult)
  } else if(this.PromiseState === 'rejected') {
    onReject(this.PromiseResult)
  } else if(this.PromiseState === 'pendding') {  // 说明可能有定时器,把方法存放起来，等待之后调用
    this.onResolveArr.push(onResolve.bind(this))
    this.onRejectArr.push(onReject.bind(this))
  }
}

new MyPromise(resolve=> {
    setTimeout(() => {
        resolve('成功')
    }, 1000)
}).then(res => console.log(res))  // 一秒后打印 成功

```


3. then是支持链式调用的，then能够链式调用的关键在于，整个then方法返回值也是一个promise，而这个返回的promise到底是成功还是失败，分为三种情况：

   a.  返回值是promise对象，返回值为成功，新的promise为成功

   b.  返回值是promise对象，返回值为失败，新的promise为失败

   c.  返回值不是promise对象，返回值为成功，值就是这个返回值

   ```js
MyPromise.prototype.then = function(onResolve, onReject) {
    // then接收两个函数，所以需要校验下
    // 1. 传递的是函数 则正常使用
    // 2. 传递的不是函数 则修改成函数
    onResolve = typeof(onResolve) === 'function' ? onResolve : val => val
    onReject = typeof(onReject) === 'function' ? onReject : reason => {throw reason}
    let thenPromise = new MyPromise((resolve, reject) => {
      const resolvePromise = cb => {
        try {
              let result = cb(this.PromiseResult)
              // 说明返回的是一个Promise对象
              if(result instanceof MyPromise) {  
                // 到底是成功还是失败，得交给then
                result.then(resolve, reject)
              } else {  // 其他值 当成成功处理
                resolve(result)
              }
        } catch(err) {  // 捕获代码的异常
            reject(err)
            throw new Error(err)
        }
      }
  
      if(this.PromiseState === 'resolved') {
          resolvePromise(onResolve)
      } else if(this.PromiseState === 'rejected') {
          resolvePromise(onReject)
      } else if(this.PromiseState === 'pendding') {  // 说明可能有定时器,把方法存放起来，等待之后调用
          this.onResolveArr.push(onResolve.bind(this))
          this.onRejectArr.push(onReject.bind(this))
      }
    })
  
    // then的返回值是一个promise, 解决了链式调用问题
    return thenPromise
  }
  
  new MyPromise((resolve, reject) => {
    resolve('第一层')
  }).then(res => {
    console.log(res)  // 第一层
    return new MyPromise((resolve, reject) => reject('第二层'))
  }).then(res => {}, err => {
    console.log(err)  // 第二层
  })
  ```



4. promise的then其实是一个微任务，也就是需要等同步任务做完在执行then

   ```js
let thenPromise = new MyPromise((resolve, reject) => {
    const resolvePromise = cb => {
      // 加一个定时器,别介意苦笑😂
      // 这样就会等待同步执行完, 再执行微任务 then
      setTimeout(() => {
        try {
          let result = cb(this.PromiseResult)
          // 说明返回的是一个Promise对象
          if(result instanceof MyPromise) {  
            // 到底是成功还是失败，得交给then
            result.then(resolve, reject)
          } else {  // 其他值 当成成功处理
            resolve(result)
          }
  
        } catch(err) {  // 捕获代码的异常
          reject(err)
          throw new Error(err)
        }
      })
      
    }

    if(this.PromiseState === 'resolved') {
      resolvePromise(onResolve)
    } else if(this.PromiseState === 'rejected') {
      resolvePromise(onReject)
    } else if(this.PromiseState === 'pendding') {  // 说明可能有定时器,把方法存放起来，等待之后调用
      this.onResolveArr.push(onResolve.bind(this))
      this.onRejectArr.push(onReject.bind(this))
    }

  })
   ```

### Promise.all的实现

- 接收一个Promise数组，数组中如有非Promise项，则此项当做成功
- 如果所有Promise都成功，则返回成功结果数组
- 如果有一个Promise失败，则返回这个失败结果

```js
class MyPromise {  
  ...
  ...
  
  static all(list) {
    const result = []
    let count = 0
    return new MyPromise((resolve, reject) => {
      const addData = (index, value) => {
        result[index] = value
        count++
        if (count === list.length) {
          resolve(result)
        }
      }

      list.forEach((item, index) => {
        if (item instanceof MyPromise) {
          item.then(res => {
            addData(index, res)
          }, err => reject(err))
        } else {
          addData(index, item)
        }
      })
    })
  }
}

let p1 = new MyPromise(resolve => {
  resolve(1)
})
let p2 = new MyPromise(resolve => {
  resolve(2)
})
let p3 = new MyPromise(resolve => {
  resolve(3)
})

MyPromise.all([p1, p2, p3]).then(res => {
  console.log(res) // [1, 2, 3]
})

```

### Promise.race的实现

- 接收一个Promise数组，数组中如有非Promise项，则此项当做成功
- 哪个Promise最快得到结果，就返回那个结果，无论成功失败

```js
class MyPromise {  
  ...
  ...
  
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      promises.forEach((promise) => {
        if (promise instanceof MyPromise) {
          promise.then(
            (res) => {
              resolve(res)
            },
            (err) => {
              reject(err)
            }
          )
        } else {
          resolve(promise)
        }
      })
    })
  }
}

let p1 = new MyPromise(resolve => {
  resolve(1)
})
let p2 = new MyPromise(resolve => {
  resolve(2)
})
let p3 = new MyPromise(resolve => {
  resolve(3)
})

MyPromise.race([p1, p2, p3]).then(res => {
  console.log(res) // 1
})
```

