---
title: 手写 axios
tags: JS
---

`axios` 是当前比较流行的发请求的一个第三方库，`axios` 支持浏览器和node端，支持 promise语法，有丰富的配置项`拦截器`，那我们来一起边使用`axios`，边手动写一下`axios`的方法吧。

<!-- more -->

```js
import axios from 'axios'

// 请求方式 1
axios({
    method: 'GET',
    url: 'http://ajax-api.itheima.net/api/settings'
}).then(res => console.log(res))

// 请求方式 2
axios.get('http://ajax-api.itheima.net/api/settings').then(res => console.log(res))
```

我们发现 `axios`发起请求有两种方式，我们来简单实现下

```js
class MyAxios {
  request(config) {
    // 这里真正发起请求,并且使用 Promise封装就好了
    return new Promise((resolve, reject) => {
      const {method = 'GET', url = '', data = {}} = config

      let xhr = new XMLHttpRequest()
      xhr.open(method, url)
      xhr.send(data)

      xhr.onreadystatechange = function() {
        if(xhr.readyState === 4 && xhr.status === 200) {
          resolve({
            status: 200,
            data: JSON.parse(xhr.responseText),
            statusText: 'OK'
          })
        }
      }
    })
  }
}


function createMyAxios (){
  let myaxios = new MyAxios()

  let request = myaxios.request.bind(myaxios)
  
  return request
}

const myaxios = createMyAxios()
// 已经实现了axios方式一
myaxios({
  method: 'GET',
  url: 'http://ajax-api.itheima.net/api/settings'
}).then(res => console.log(res))
```

`myaxios` 已经能够直接调用发起请求了。

那么方式二怎么解决呢，并且除了`get`,还有其他比如`post`、`delete`等请求，我们可以把请求方法收集成一个数组，然后循环遍历依次创建请求就好。

```js
let methodList = ['get', 'delete', 'head', 'options', 'put', 'post', 'patch']

methodList.forEach(method => {
  MyAxios.prototype[method] = function(...arg) {
    if(['get','delete','head','options'].includes(method)) {
      return this.request({
        method: method,
        url: arg[0]
      })
    } else if(['put', 'post', 'patch'].includes(method)) {
      return this.request({
        method: method,
        url: arg[0],
        data: arg[1]
      })
    }
  }
})
```

我们已经给 MyAxios的原型对象挂载好方法了， 但是我们导出的`request` 是没有这些方法的，所以有必要把原型对象下的方法，复制到`request`里，所以我们封装一个方法。

```js
function extendFn(target, source, context) {
  for(let key in source) {
    if(source.hasOwnProperty(key)) {
      if(typeof source[key] === 'function') {
        // 这里注意, 函数可能有this不能直接赋值
        // 所以把当前上下文环境 绑定过来
        target[key] = source[key].bind(context)
      } else {
        target[key] = source[key]
      }
    }
  }
}
```

函数封装好了，我们调用函数，把原型对象下的方法，复制到`request`函数对象里

```js
function createMyAxios() {
  let myaxios = new MyAxios()

  let request = myaxios.request.bind(myaxios)

  // 原型对象方法 复制到 request函数对象中
  extendFn(request, MyAxios.prototype, myaxios)
  
  return request
}
```

这样，我们终于可以用第二种方式发请求了, 完整代码如下

```js
class MyAxios {
  request(config) {
    // 这里真正发起请求,并且使用 Promise封装就好了
    return new Promise((resolve, reject) => {
    
      const {method = 'GET', url = '', data = {}} = config

      let xhr = new XMLHttpRequest()
      xhr.open(method, url)
      xhr.send(data)

      xhr.onreadystatechange = function() {
        if(xhr.readyState === 4 && xhr.status === 200) {
          resolve({
            status: 200,
            data: JSON.parse(xhr.responseText),
            statusText: 'OK'
          })
        }
      }
    })
  }
}

let methodList = ['get', 'delete', 'head', 'options', 'put', 'post', 'patch']

methodList.forEach(method => {
  MyAxios.prototype[method] = function(...arg) {
    if(['get','delete','head','options'].includes(method)) {
      return this.request({
        method: method,
        url: arg[0]
      })
    } else if(['put', 'post', 'patch'].includes(method)) {
      return this.request({
        method: method,
        url: arg[0],
        data: arg[1]
      })
    }
  }
})

function extendFn(target, source, context) {
  for(let key in source) {
    if(source.hasOwnProperty(key)) {
      if(typeof source[key] === 'function') {
        // 这里注意, 函数可能有this不能直接赋值
        // 所以把当前上下文环境 绑定过来
        target[key] = source[key].bind(context)
      } else {
        target[key] = source[key]
      }
    }
  }
}

function createMyAxios() {
  let myaxios = new MyAxios()

  let request = myaxios.request.bind(myaxios)

  extendFn(request, MyAxios.prototype, myaxios)
  
  return request
}

const myaxios = createMyAxios()

myaxios.get('http://ajax-api.itheima.net/api/settings').then(res => console.log(res))

```



