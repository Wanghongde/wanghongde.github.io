---
title: Flex布局
tags: CSS
---

## Flex 的容器和项目

随着主流框架的流行，人们已经渐渐抛弃了低版本浏览器了，flex 布局也成为了主流布局方式。

现在的 flex布局 配合 `vw`和`vh` 也已经成为了移动端的主流布局方式, 这种移动端适配方案我们之后再聊。

现在先主要聊一下flex相关内容。

flex 是 `flexible box` 的缩写， 意为弹性布局，可以让我们布局页面更加灵活。

任何元素都可以做为 flex元素(块元素、甚至行内元素)，采用flex布局的元素，我们可以叫他`容器`，而`容器`里的元素称为`项目`， `.box` 就是容器，`.item`就是项目。
<!-- more -->

```html
<style>
    .box {
        display: flex;
        width: 200px;
        height: 200px;
        border: 1px solid red;
    }
    .item {
        width: 20px;
        height: 20px;
        background-color: aquamarine;
        border-radius: 50%;
    }
</style>  

<span class="box">
    <span class="item"></span>
</span>
```

## Flex 容器的属性

容器默认存在两个轴线，默认水平的叫`主轴`，垂直的叫`交叉轴`也可以叫`侧轴`，当然轴线是可以修改的，项目默认沿着主轴排列。

flex容器有六个属性:

```
flex-direction // 控制主轴的方向 
flex-wrap  // 是否可以换行
justify-content // 主轴的对齐方式
align-items  // 侧轴的对齐方式
align-contents // 主轴存在多根，才生效

flex-flow: flex-direction flex-wrap // 复合属性 flex-flow:主轴方向 换行

```

## Flex 项目的属性

flex项目也有六个属性

```
order //顺序 控制项目从左往右的排列顺序 默认是0，可以设置负数，越小越靠左
flex-grow // 放大 控制项目 在剩余空间放大  0不放大  1放大   默认不放大 0
flex-shrink // 缩小 控制项目在空不足缩小   0不缩小  1缩小   默认可缩小 1
flex-basis // 分配剩余空间前 项目占据的大小 	默认是 auto 元素本身的大小 可设置 px、%等单位
align-self // 设置项目本身的 侧轴对齐方式 可覆盖align-items

flex: flex-grow flex-shrink flex-basis // 复合属性

```

## Flex 使用

使用flex简单实现个圣杯布局

```html
<style>
    body {
        margin: 0;
    }
    .container {
        display: flex;
        min-height: 100vh;
        flex-direction: column;
    }
    header{
        background: pink;
    }
    footer{
        background: #ccc;
    }
    .container-body-content {
        background: chocolate;
    }
    .container-body-nav {
        background: aquamarine;
    }
    .container-body-aside {
        background: thistle;
    }


    header,
    footer {
        flex: 1;
    }

    .container-body {
        display: flex;
        align-self: center;
        flex: 1;
        width: 80vw;
        min-height: 700px;
    }

    .container-body-content {
        flex: 1;
    }

    .container-body-nav,
    .container-body-aside {
        flex: 0 0 12em;
    }

    .container-body-nav {
        order: -1;
    }
</style>

<div class="container">
    <header>头部</header>
    <div class="container-body">
        <main class="container-body-content">主要内容 我是很多内容</main>
        <nav class="container-body-nav">nav</nav>
        <aside class="container-body-aside">侧边栏</aside>
    </div>
    <footer>底部</footer>
</div>
```

