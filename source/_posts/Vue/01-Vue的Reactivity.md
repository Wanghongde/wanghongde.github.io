---
title: Vue的响应式
tags: Vue
---
### 响应式是什么

什么是响应式呢，举个简单例子 

```js
let a = 1

let b = a * 10  // 我们希望 b 是 a 的 10 倍
console.log(b)  // 10

a = 5
console.log(b)  // 我们还希望 b 是 a 的 10 倍,那这个时候就麻烦了,需要接着计算 
```

假设有一个函数`fn` ，当 `a` 改变则函数`fn`自动执行，将`b`重新计算就好了，这其实就是可以简单理解成`响应式`。

说的更准确点，提前准备一个函数`watchEffect`，当函数里面的依赖项(数据)发生更改，就自动执行`watchEffect`，这就是`响应式` 。

<!-- more -->

我们再进一步，函数里的依赖项发生改变，则更新页面，不就是`model`修改，`view`自动更新嘛。

`@nx-js/observer-util` 刚好就提供了类似的功能，只不过函数名叫`observe`。

```js
import { observable, observe } from '@nx-js/observer-util'

let obj = observable({
  age: 18
})

observe(() => {  // 每当数据改变 observe函数 就自动执行
  console.log(obj.age)
})

obj.age += 1  // 当 obj.age 增加就会执行observe  19

obj.age += 1  // 当 obj.age 增加就会执行observe  20
```

理解了`响应式` 是什么，那接下来我们也一步步实现个响应式。

### 发布订阅模式

发布订阅模式是设计模式的一种，说个现实的例子，小刘、小红、小张去电脑城买电脑，但是没有优惠，于是给店员留了电话到电话本上，店员等电脑城搞活动，打电话通知这三个人。店员就是`发布者`，记录了电话的电话本就是`缓存列表`，消费者就是`订阅者` 。

给DOM添加事件的`addEventListener`也是发布订阅。

接下来我们模拟着实现一下

```js
// 暂时存放 watchEffect 的变量
let activeEffect

class Dep {
    // 用 Set 做订阅者列表 以防止列表中添加多个完全相同的函数
    subscribers = new Set()
	
	// 依赖收集方法
	depend() {
        // 如果 activeEffect 为空 则代表没有 watcheffect
        if(activeEffect) {
            this.subscribers.add(activeEffect)
        }
    }

	// 更新操作 通常在值修改后执行 发布消息
	notify() {
        this.subscribers.forEach(effect => {
            effect()
        })
    }
}

// 模仿 Vue3 的 watchEffect 函数
function watchEffect(effect) {
    // 先把传进来的函数放到 activeEffect
    activeEffect = effect
    // 执行一下 watchEffect 里面的函数
    effect()
}

// 使用
let dep = new Dep()
// 暂定一个全局变量 age
let globalAge = 18

const user = {
    get age() {
        dep.depend()
        return globalAge
    },
    set age(newAge) {
        globalAge = newAge
        dep.notify()
    }
}

watchEffect(() => {
    console.log(user.age)
})

user.age += 1
```

我们初步实现了最初的想法， 每当依赖项`user.age` 修改，都会执行`watchEffect`函数。

而这里的 dep对象 就是`发布者`,  subscribers 就是`缓存列表` ，dep.notify() 是 `发布消息`。

### 代理模式

虽然上边实现了功能，但是用起来太麻烦了，需要提供对象的 `getter`和`setter`，还要提供手动的执行依赖收集函数，这是不方便的，所以需要接着封装。

封装前先简单聊一下`代理模式`， 代理简单说就是，我们需要租房，而我们是很难找到房源的，房源都在中介手里，这个中介就是代理，他给我们介绍什么房子，我们就拿到什么房子。

那我们来一起简单实现下：

```js
let activeEffect

// ========= start 这里跟刚才一样 ===========
class Dep {
    subscribers = new Set()
	
	depend() {
        if(activeEffect) {
             this.subscribers.add(activeEffect)
        }  
    }

	notify() {
        this.subscribers.forEach(effect => effect())
    }
}

function watchEffect(effect) {
    activeEffect = effect
    effect()
}

// ========= end ===========

// 模仿 Vue3 的 reactive
function reactive(raw) {
    // 遍历对象上存在的 key
    Object.keys(raw).forEach(key => {
        // 为每个 key 都创建一个依赖对象
        let dep = new Dep()
        
        let realValue = raw[key]
        
        // 用 get 和 set 重写原对象的key
        Object.defineProperty(raw, key, {
            // 在 get 和 set 里调用依赖对象的对应方法
            get() {
                dep.depend()
                return realValue
            },
            set(newVal) {
                realValue = newVal
                dep.notify()
            }
        })
    })
    
    return raw
}

// 使用
let obj = reactive({ age: 18 })
let fn = () => { console.log(obj.age) }
watchEffect(fn)

obj.age += 1  // 每当修改 age 都会执行 fn 19

obj.age += 1  // 每当修改 age 都会执行 fn 20
```

这里的 `Object.defineProperty`就是做的`代理模式` ，现在在使用方式上基本和 Vue3 一样了。

但还不是最终版本，vue3已经放弃了 `IE`， 并且`Object.defineProperty	` 在监听对象时，新增的数据是监听不到的。

所以使用了 `Proxy` 来做代理。

### Proxy版

使用了发布订阅+代理模式实现基本的`reactivity`。

```js
// 定义一个暂时存放 watchEffect 传进来的参数的变量
let activeEffect

// 这个Dep应该是Dependence的缩写，意为依赖。实际上就相当于发布-订阅模式中的发布者类
// 定义一个 Dep 类，该类将会为每一个响应式对象的每一个键生成一个发布者实例
class Dep {
  // 用 Set 做缓存列表以防止列表中添加多个完全相同的函数
  subscribers = new Set()

  // 构造函数接受一个初始化的值放在私有变量内
  constructor(value) {
    this._value = value
  }

  // 当使用 xxx.value 获取对象上的 value 值时
  get value() {
    // 代理模式 当获取对象上的value属性的值时将会触发 depend 方法
    this.depend()

    // 然后返回私有变量内的值
    return this._value
  }

  // 当使用 xxx.value = xxx 修改对象上的 value 值时
  set value(value) {
    // 代理模式 当修改对象上的value属性的值时将会触发 notify 方法
    this._value = value
    // 先改值再触发 这样保证触发的时候用到的都是已经修改后的新值
    this.notify()
  }

  // 这就是我们常说的依赖收集方法
  depend() {
    // 如果 activeEffect 这个变量为空 就证明不是在 watchEffect 这个函数里面触发的 get 操作
    if (activeEffect) {
      // 但如果 activeEffect 不为空就证明是在 watchEffect 里触发的 get 操作
      // 那就把 activeEffect 这个存着 watchEffect 参数的变量添加进缓存列表中
      this.subscribers.add(activeEffect)
    }
  }

  // 更新操作 通常会在值被修改后调用
  notify() {
    // 遍历缓存列表里存放的函数 并依次触发执行
    this.subscribers.forEach((effect) => {
      effect()
    })
  }
}
// 模仿 Vue3 的 watchEffect 函数
function watchEffect(effect) {
  // 先把传进来的函数放入到 activeEffect 这个变量中
  activeEffect = effect
  // 然后执行 watchEffect 里面的函数
  effect()
  activeEffect = null
}

// 定义一个 WeakMap 数据类型 用于存放 reactive 定义的对象以及他们的发布者对象集
const targetToHashMap = new WeakMap()

// 定义 getDep 函数 用于获取 reactive 定义的对象所对应的发布者对象集里的某一个键对应的发布者对象
function getDep(target, key) {
  // 获取 reactive 定义的对象所对应的发布者对象集
  let depMap = targetToHashMap.get(target)
  // 如果没获取到的话
  if (!depMap) {
    // 就新建一个空的发布者对象集
    depMap = new Map()
    // 然后再把这个发布者对象集存进 WeakMap 里
    targetToHashMap.set(target, depMap)
  }

  // 再获取到这个发布者对象集里的某一个键所对应的发布者对象
  let dep = depMap.get(key)

  // 如果没获取到的话
  if (!dep) {
    // 就新建一个发布者对象并初始化赋值
    dep = new Dep(target[key])

    // 然后将这个发布者对象放入到发布者对象集里
    depMap.set(key, dep)
  }
  // 最后返回这个发布者对象
  return dep
}

// 模仿 Vue3 的 reactive 函数
function reactive(obj) {
  // 返回一个传进来的参数对象的代理对象 以便使用代理模式拦截对象上的操作并应用发布-订阅模式
  return new Proxy(obj, {
    // 当触发 get 操作时
    get(target, key) {
      // 先调用 getDep 函数取到里面存放的 value 值
      const value = getDep(target, key).value
      // 如果 value 是对象的话
      if (value && typeof value === 'object') {
        // 那就把 value 也变成一个响应式对象
        return reactive(value)
      } else {
        // 如果 value 只是基本数据类型的话就直接将值返回
        return value
      }
    },
    // 当触发 set 操作时
    set(target, key, value) {
      // 调用 getDep 函数并将里面存放的 value 值重新赋值成 set 操作的值
      getDep(target, key).value = value
    }
  })
}

const obj = reactive({
  age: 18
})

watchEffect(() => {
  console.log(obj.age)
})

obj.age++
```





