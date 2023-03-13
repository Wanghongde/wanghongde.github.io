---
title: Grid布局
tags: CSS
---

Grid布局既网格布局，是一种新的 CSS 的布局模式，是比较强大的布局方式。

Flex布局也就是弹性布局很好用，但是只能解决`一维布局`，也就是一行或者一列进行布局。

Grid布局则是网格布局，可以解决`二维布局`，也就是由纵横相交的两组网格线形成的框架性布局结构，能够同时处理行与列 。

那在布局的时候，我们能用Flex布局就是用Flex布局解决，解决不了则使用 Grid布局。

<!--more-->

Grid布局属性可以分为两大类

- 容器属性
  - display: grid / inline-grid
  - grid-template-rows:  显式网格分几行  分别是多高
  - grid-template-columns:  显式网格分几列  分别是多宽
  - grid-rows-gap / grid-columns-gap / grid-gap: 网格的间隙大小
  - justify-item / align-item / place-item: 项目相对单元格 水平/垂直的 位置
  - justify-content / align-content / place-content: 整个项目块在容器的 水平/垂直的位置
  - grid-auto-flow: 网格的排列顺序 默认是row既 `先行后列`, 也可以改成column既`先列后行`
  - grid-auto-rows: 隐式网格的高
  - grid-auto-columns： 隐式网格的宽 
  - grid-template-areas: 定义区域，一个区域由一个或者多个单元格组成
- 项目属性
  -  gird-area: 指定项目放在哪一个区域，需要与容器的 grid-template-areas搭配
  - justify-self：指定单个项目 相对单元格 水平 的位置
  - align-self： 指定单个项目 相对单元格垂直的位置
  - place-self:  指定单个项目对象单元格 水平/垂直的位置
  - grid-column-start：左边框所在的垂直网格线
  - grid-column-end：右边框所在的垂直网格线
  - grid-row-start：上边框所在的水平网格线
  - grid-row-end：下边框所在的水平网格线