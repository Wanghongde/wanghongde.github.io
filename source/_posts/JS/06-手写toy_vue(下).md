---
title: 手写 toy_vue (下)
tags: JS
---

我们已经可以将数据编译到页面上，但是数据改变不能影响页面的从新编译，所以要在数据修改时，也就是`setter`中，进行事件的发布，在`getter`时进行事件的监听。这里肯定需要用到`发布订阅`，所以先抛开其他，我们先实现一个`发布订阅`

```js
function Dep() {
  this.list = []
}

Dep.prototype.addSub = function(sub) {
  this.list.push(sub)
}

Dep.prototype.notify = function() {
  this.list.forEach(sub => {
    sub.update()
  })
}

function Watcher(fn) {
  this.fn = fn
}

Watcher.prototype.update = function() {
  this.fn()
}
```

<!--more-->

当我们编译页面时，应该实例化一个Watcher,并将实例缓存到`list`列表中

```js
function Compile() {
    if(node.nodeType === 3 && reg.test(node.textContent)) {
        let text = node.textContent  // 缓存一下最初的内容
     // ...
     new Watcher(function(newVal) {
         node.textContent = text.replace(reg, newVal)  // 这里要拿到修改后的新数据
     })
 	}
} 
```

这里需要注意，在调用watcher的函数时，需要传入新修改的值，所以这里要想办法让`update`里能计算出最新的值，所以在实例化的时候，要把计算前需要的数据传递到watcher实例下:

```js
function Compile() {
    if(node.nodeType === 3 && reg.test(node.textContent)) {
     // ...
        
     // vm: vue实例 有所有的数据
     // RegExp.$1: 提取分组的数据的key  
     new Watcher(vm, RegExp.$1,function(newVal) {
         node.textContent = node.textContent.replace(reg, newVal)  // 这里要拿到修改后的新数据
     })
 	}
} 
```

那我们还需要改造`Watcher`构造函数了，这样`update`就可以通过`this`访问到数据了

```js
function Watcher(vm, expression, fn) {
  this.vm = vm
  this.expression = expression
  this.fn = fn
  
}
```

这里我们不仅仅需要处理字段，还应该将这一次的 `watcher实例` 存储到`list`缓存列表，这样之后修改数据，才能执行`watcher实例`的`update`函数

```js
function Watcher(vm, expression, fn) {
  this.vm = vm
  this.expression = expression
  this.fn = fn

  Dep.target = this

  let val = vm
  expression.split('.').forEach(key => val[key])  // 这里在读取数据所以回触发数据的getter

  Dep.target = null

}
```

由于我们在`val[key]`的时候会读取数据，所以会触发`getter`，那么就可以将 `Dep.target`,也就是`watcher实例` 存到`list`缓存列表,，在设置值的时候我们去触发`notify`，让缓存列表的`update`都执行。

```js
function Observe(data) {
  let dep = new Dep()
  for(let key in data) {
    let value = data[key]
    observe(value)

    Object.defineProperty(data, key, {
      enumerable: true,
      get() {
        Dep.target && dep.addSub(Dep.target)

        return value
      },
      set(newVal) {
        if(value === newVal) return 

        value = newVal

        observe(newVal)
        dep.notify() 
      }
    })
  }
}
```

到这里，我们已经可以修改内容，并实现页面更新了，但是`v-model`我们还没处理，当`nodeType`是1,并且有`v-model`属性我们也需要进行数据的缓存，以响应数据，我们还需要注册`input`事件，监听输入内容，同步到`model`数据里。

```js
function Compile() {
    // ...
    if(node.nodeType === 1) {
       Array.from(node.attributes).forEach(item => {
           if(item.name === 'v-model') {
               let arr = item.textContent.split('.')
               let val = vm

               arr.forEach((key, index) => {
                   if(index === arr.length - 1) {
                       new Watcher(vm, item.textContent, function(newVal) {
                           node.value = val[key]
                       })
                       return node.value = val[key]
                   }
                   val = val[key]
               })

               node.addEventListener('input', e => {
                   let val = vm
                   arr.forEach((key, index) => {
                       if(index === arr.length - 1) {
                           return val[key] = e.target.value
                       }
                       val = val[key]
                   })
               })

           }
       })
   }
}  
```

这样我们就彻底完成了一个`toy_vue`。