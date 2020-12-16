---
layout: post
title: CSS实现0.5px边框
date: 2020-12-16 21:35:20
tags:
- css
- border
categories: 
- 前端
---

html结构如下

```html
<div class="demo"></div>
```

## 单边框

### border + border-image + linear-gradient

```css
.demo {
 	height: 100px;
    width: 100px;
    background-color: green;
    background-clip: content-box;
    border-bottom: 1px solid transparent;
    border-image: linear-gradient(to bottom,red 50%,transparent 50%) 0 0 100%;
}
```
使用这种方法有一个问题，必须设置background-clip为content-box，不然边框颜色设置了透明，会显示div的背景颜色（如果没有显示背景颜色没有问题）。但因为设置了background-clip，如果再设置padding，padding部分就不会显示背景颜色，有弊端。

### 伪元素 + background-image + linear-gradient

```css
.demo {
    height: 100px;
    width: 100px;
    background-color: green;
    position: relative;
}
.demo::after {
    content: "";
    position: absolute;
    left: 0;
    bottom: -1px;
    width: 100%;
    height: 1px;
    background-image: linear-gradient(to bottom,red 50%, transparent 50%);
}
```

以上方法只是改变渐变颜色显示部分，以实现显示上的0.5px

### 伪元素 + background-color + transform

```css
.demo {
    height: 100px;
    width: 100px;
    background-color: green;
    position: relative;
}
.demo::after {
    content: "";
    position: absolute;
    left: 0;
    bottom: -1px;
    width: 100%;
    height: 1px;
    background-color: red;
    transform-origin: 0 0;
    transform: scaleY(0.5)
}
```

通过缩放实现0.5px的边框。

## 多边框

### 伪元素+ border + transfrom

```css
.demo {
    height: 100px;
    width: 100px;
    background-color: green;
    position: relative;
}
.demo::after {
    content: "";
    position: absolute;
    width: 200%;
    height: 200%;
    border: 1px solid red;
    transform-origin: 0 0;
    transform: scale(0.5) translate(-1px,-1px)
}
```

