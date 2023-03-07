---
title: Vue开发移动端css适配
tags: Vue
---

相对于 PC端来说，移动端设备在进行css适配时要麻烦一些。

这时候我们就应该理顺出一套解决方法，适配各种手机。

之前呢，我们使用`rem`配合手淘的`lib-flexible`实现移动端适配。

现在有了更好的`vw`和`vh`单位，再配合弹性布局和`grid`布局。

`vw` 视口的最大宽度，`1vw` = 视口宽度的百分之一

`vh` 视口的最大高度， `1vh` = 视口高度的百分之一

<!--more-->

假如我们是 `750px` 宽的设计稿，那我们只需要量出元素的宽度，比如元素宽度占屏幕一半`375px`，然后让`375px` 自动转成`50vw`，那这样就既让`750px`的屏幕占一半的宽，其他分辨率下元素也是占屏幕宽度一半，这不就完成适配了嘛。

那如何让`px`转成`vw`呢？

### px-to-vw

如果是开发多页面应用，并不借助框架或者构建工具，可以使用`vs code` 的插件`px-to-vw`

只需要在插件里搜索并安装，插件默认设计稿是`750px`，也就是`7.5px`会转换成`1vw`。

 插件不会在你写完`px`自动转换成`vw`，需要你选中希望转换的代码，按下`alt +z`，进行转换。

###  postcss-px-to-viewport 

如果开发的是单页面应用，使用了框架和构建工具，就可以借助`postcss`的插件` postcss-px-to-viewport `

```
npm install postcss-px-to-viewport
```

然后在`vite.config.ts`或者`webpack.config.js`或者`vue.cofnig.js`配置文件

```js
import postcsspxtoviewport from 'postcss-px-to-viewport'

export default defineConfig({
  // ...  
  css: {
    postcss: {
      plugins: [
        postcsspxtoviewport({
          unitToConvert: 'px', // 要转化的单位
          viewportWidth: 750 // UI设计稿的宽度
        })
      ]
    }
  }
})
```

我们在页面开发时直接写入`px`单位就好，在经过构建工具打包后的代码里，会自动转换成`vw`单位。



