---
title: 手写 toy_vue (上)
tags: JS
---

vue 是现在比较主流的基于`MVVM`的框架，当数据改变页面就更新，页面更新数据也相应改变。

```html
<div id="app">
    <h1>
        {{ title }}
    </h1>
    <input type="text" v-model="info.address" />
</div>

<script src="vue.js"></script>
<script>
	let app = new Vue({
        el: 'app',
        data: {
            title: '使用vue',
            info: {
                address: '石家庄'
            }
        }
    })
</script>
```

那么`vue` 是如何做到将data的数据渲染到页面，又是如何实现双向绑定呢，我们来一起做一下。

<!--more-->

首先创建`Vue`构造函数，然后进行data`、`options`等参数的处理

另外，数据的修改需要做页面的更新，所以数据需要劫持，这样当数据修改的时候我们可以做页面更新等一系列的操作

```js
function Vue(options = {}) {
  this.$options = options
  let data = this._data = options.data

  observe(data)
}

function Observe(data) {
  for(let key in data) {
    let value = data[key]
    observe(value)

    Object.defineProperty(data, key, {
      enumerable: true,
      get() {
        return value
      },
      set(newVal) {
        if(value === newVal) return 

        value = newVal

        observe(newVal)
      }
    })
  }
}

function observe(data) {
  if(typeof data !== 'object') return 
  new Observe(data)
}
```

我们在使用vue时，可以直接`app.title`进行数据修改，所以我们有必要做一个代理，把data对象的数据，代理到`app实例`下

```js
function Vue(options = {}) {
  // ...

  for(let key in data) {
    Object.defineProperty(this, key, {
      enumerable: true,
      get() {
        return data[key]
      },
      set(newVal) {
        if(data[key] === newVal) return 
        data[key] = newVal
      }
    })
  }
}
```

我们已经可以对数据进行修改，并且操作数据，会有相应的`getter`和`setter`了。

下一步我们需要把页面内容进行渲染，当碰到双大括号，我们利用正则进行替换，并且这里为了减少页面回流，可以使用文档碎片接受页面内容并进行处理

```js
function Vue(options = {}) {
  // ...

  new Compile(options.el, this)
 
}


function Compile(el, vm) {
  let app = document.querySelector(el)
  let fragment = document.createDocumentFragment()
  let reg = /\{\{\s*(\S*)\s*\}\}/g

  while(first = app.firstChild) {
    fragment.appendChild(first)
  }

  replace(fragment)

  function replace(fragment) {
    Array.from(fragment.childNodes).forEach(node => {
      if(node.nodeType === 3 && reg.test(node.textContent)) {
        let arr = RegExp.$1.split('.')
        let val = vm
        arr.forEach(key => val = val[key])

        node.textContent = node.textContent.replace(reg, val)
      }
  
      if(node.childNodes) {
        replace(node)
      }
    })
  }

  app.append(fragment)
}
```

这里已经可以将双大括号也就是小胡子语法进行替换了，但是`v-model`还没有处理，首先得是一个标签也就是`nodeType` 是1，然后标签身上得有`v-bind`属性，我们就可以给这个标签设置value了(当然这里暂时不考虑组件)。

```js
function replace(fragment) {
    Array.from(fragment.childNodes).forEach(node => {
      // ...
 
      if(node.nodeType === 1) {
        Array.from(node.attributes).forEach(item => {
          if(item.name === 'v-model') {
            let arr = item.textContent.split('.')
            let val = vm

            arr.forEach((key, index) => {
              if(index === arr.length - 1) {
                return  node.value = val[key]
              }
              val = val[key]
            })
            
          }
        })
      }
      if(node.childNodes) {
        replace(node)
      }
    })
  }
```

至此我们已经实现了最基本的`model`,渲染到`view`上了。

但是我们修改数据`model`，并没有更新页面，这里就要使用`数据劫持`配合我们的`发布订阅模式`了。

