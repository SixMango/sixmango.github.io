---
layout: post
title: 浏览器知识随笔
date: 2020-12-02 22:26:43
tags:
- browser
- chrome
categories: 
- 前端
---

## 浏览器的构成

浏览器简单结构可以分为用户界面、浏览器引擎、渲染引擎（常被称为浏览器内核）。用户界面用于展示除标签页窗口之外的其他用户界面内容。渲染引擎负责渲染用户请求的页面内容。在用户界面和渲染引擎之间有个浏览器引擎，用于在用户界面和渲染引擎之间传递数据。

渲染引擎下面还有很多小的功能模块，比如负责网络请求的网络模块，用于解析和执行js的js解释器，解析HTML用到的XML解析器，用于绘制基本的浏览器窗口内控件的UI后端，还有数据存储持久层，帮助浏览器保存各种数据，比如cookie等等。
![](https://ae01.alicdn.com/kf/U470190e3982246ec89d2e71a06bcfe2ar.jpg)

## 浏览器内核

不同浏览器使用的内核也不近相同，下表做了个总结

| 浏览器  | 内核    | 备注                                                         |
| ------- | ------- | :----------------------------------------------------------- |
| IE      | Trident | IE、猎豹安全、360极速浏览器、百度浏览器                      |
| Firefox | Gecko   | Mozila自主研发的渲染引擎                                     |
| Safari  | Webkit  | Safari推出时使用的渲染引擎，后来将webkit开源                 |
| Chrome  | Blink   | Chromium 项目（基于webkit，同时也开源）中研发的新一代渲染引擎 |
| Opera   | Blink   | 最初使用自主研发的Presto（最后版本12.17），后转用Chromium 项目 |
| Edge    | Blink   | 最初使用自主研发的EdgeHTML，后转用Chromium 项目（2020年四月） |

## Chrome

### 结构

Chrome浏览器是一个多进程结构应用，根据服务功能不同可以拆分成浏览器进程（负责与浏览器其他进程协调工作）、网络进程（负责发起接受网络请求）、GPU进程（负责图形渲染）、插件进程（负责控制网站使用的插件，如flash）、缓存进程（负责数据持久化）、渲染进程（负责渲染Tab标签内的内容）等等。默认情况下浏览器会为每个标签页都创建一个进程，因为chrome默认使用的进程模型是Process-per-site-instance，除此之外还有三种模型，详情参考https://www.chromium.org/developers/design-documents/process-models。

> 进程（Process）是操作系统进行资源分配和调度的基本单元，可以申请和拥有计算机资源，进程是程序的基本执行实体。
> 线程（Thread）是操作系统能够进行运算调度的最小单位，一个进程中可以并发多个线程，每条线程并行执行不同的任务。
> 当启动某个程序时，就会创建一个进程来执行任务代码，同时会为该进程分配内存空间，该应用程序的状态都保存在该内存空间里。当应用关闭时，该内存空间就会被回收。进程可以启动更多的进程来执行任务，由于每个进程分配的内存空间是独立的，如果两个进程间需要传递某些数据，则需要通过进程间通信管道IPC来传递。很多应用程序都是多进程的结构，这样是为了避免某一个进程卡死，由于进程间相互独立，这样不会影响到整个应用程序。

Chrome在不同硬件条件的设备上，会整合一些服务进程到同一进程，以达到更好的表现效果。
![](https://ae01.alicdn.com/kf/Ud8ecc32ebc9a4ea998ed64236fa3c112K.jpg)

### 导航

导航栏的工作是由浏览器进程控制，浏览器进程又对这些工作进一步划分，使用不同线程进行处理：

- ui thread：控制浏览器上的按钮及输入框
- network thread：处理网络请求，从网上获取数据
- storage thread：控制文件等的访问

当在浏览器地址栏中输入内容获得页面内容时，浏览器大致可以分为以下几步：
1. 处理输入

   ui thread判断用户输入的是url还是query

2. 开始导航

   如果输入的是url，ui thread会启动一个network thread来获取数据。如果输入是query，就会使用默认配置的搜索引擎来查询，同时ui thread控制tab展现spinner表示正在加载中。

   network thread会执行DNS查询，随后为请求建立TLS连接。

   如果network thread接收到了重定向请求头如301，network thread会通知ui thread服务器要求重定向，之后另外一个url请求会被触发。

3. 读取响应

   当请求响应返回的时候，network thread会依据Content-Type及MIME Type判断响应内容的格式，如果响应内容的格式是HTML ，会把这些数据传递给renderer process。

   Safe Browsing检查也会在此时触发，如果域名或者请求内容匹配到已知的恶意站点，network thread会展示一个警告页。

4. 查找渲染进程

   network thread确认浏览器可以导航到请求网页，network thread会通知ui thread数据已经准备好，ui thread会查找到一个renderer process进行网页的渲染。

   > ui thread可以在发送url请求给network thread时，并行预先查找和启动一个渲染进程，以提高效率。

5. 确认导航

   browser process会给renderer process发送IPC消息来确认导航，一旦browser process收到renderer process的渲染确认消息，导航过程结束，页面加载过程开始，此时地址栏会更新，历史也会更新，可通过返回键返回页面。

   直到渲染结束，renderer process会通知browser process，ui thread会停止展示tab中的spinner。

### 渲染

渲染进程的核心目的在于转换 HTML CSS JS 为用户可交互的 web 页面。渲染进程中主要包含以下线程:

- 主线程 Main thread
- 合成器线程 Compositor thread
- 栅格线程 Raster thread

![](https://ae01.alicdn.com/kf/Ud5a1b6f3c01741e3b467491039acb867L.jpg)

浏览器渲染过程：

1. 构建 DOM

   主线程对HTML进行Tokeniser标记化，通过词法分析，将输入HTML内容解析成多个标记，根据识别后的标记进行dom tree构造，在dom tree构造过程中会创建document对象，然后以document为根节点的dom tree不断进行修改，向其中添加各种元素。

   渲染html为DOM的方法由HTML Standard定义。

2. 加载次级的资源

   网页中常常包含诸如图片，CSS，JS等额外的资源，这些资源需要从网络上或者cache中获取。主进程可以在构建DOM的过程中会逐一请求它们，为了加速preload scanner会同时运行，如果在html中存在img，link等标签，preload scanner会把这些请求传递给browser process中的network thread进行相关资源的下载。

3. JS 的下载与执行

   当遇到script标签时，主线程会停止解析HTML，而去加载，解析和执行JS代码，停止 解析html的原因在于JS可能会改变dom tree（使用诸如 document.write()等API）。
   
   因此要把script标签放在合适的位置，或者使用async或defer属性来异步加载执行js。

4. 样式计算

   仅仅渲染dom tree还不足以获知页面的具体样式，主进程还会基于CSS选择器解析CSS获取每一个节点的最终的计算样式值。即使不提供任何CSS，浏览器对每个元素也会有一个默认的样式。

5. 获取布局

   想要渲染一个完整的页面，除了获知每个节点的具体样式，还需要获知每一个节点在页面上的位置，布局是找到所有元素的几何关系的过程。

   通过遍历DOM及相关元素的计算样式，主线程会构建出包含每个元素的坐标信息及盒子大小的layout tree。layout tree和dom tree类似，但是其中只包含页面可见的元素，如果一个元素设置了display:none ，这个元素不会出现在layout tree上，伪元素虽然在dom tree上不可见，但是在layout tree上是可见的。

6. 绘制各元素

   即使知道了不同元素的位置及样式信息，我们还需要知道不同元素的绘制先后顺序才能正确绘制出整个页面。在绘制阶段，主线程会遍历layout tree以创建绘制记录。绘制记录可以看做是记录各元素绘制先后顺序的笔记。

7. 合成帧

   复合（compositing）是一种分割页面为不同的图层，并单独栅格化，随后组合为帧的技术。不同层的组合由合成器线程完成。

   主线程会遍历layout tree来创建layer tree，添加了will-change CSS属性的元素，会被看做单独的一层。

   一旦layer tree被创建，渲染顺序被确定，主线程会把这些信息传递给合成器线程，合成器线程会栅格化每一层。有的层的可以达到整个页面的大小，因此合成器线程将它们分成多个图块，并将每个图块发送给栅格线程，栅格线程会栅格化每一个并存储在GPU显存中。
   
   ![](https://ae01.alicdn.com/kf/U9f8a9d38dd6e414e9733dbc591a0cad7f.jpg)
   
   一旦磁贴被光栅化，合成器线程会收集称为“drow quards”的图块信息以创建合成帧。

   合成帧随后会通过IPC消息传递给浏览器进程，由于浏览器的UI改变或者其它拓展的渲染进程也可以添加合成帧，这些合成帧会被传递给GPU用以展示在屏幕上，如果滚动发生，合成器线程会创建另一个合成帧发送给GPU。
   
   ![](https://ae01.alicdn.com/kf/U7a357d789bbe4db282a07e38b3269c590.jpg)

**重排**：当改变元素尺寸位置属性时，会重新进行样式计算，布局和绘制
**重绘**：当改变元素颜色属性时，只会发生样式计算和绘制
如果在运行动画时，还有大量的js任务需要执行，因为布局绘制和js的执行都是在主线程运行的，当在一帧的时间内，布局和绘制结束后，还有剩余时间，js就会拿到主线程的使用权，如果js执行时间过长就会导致在下一帧开始时，js没有及时归还主线程，导致下一帧动画没有按时渲染，就会出现页面动画的卡顿。我们可以使用俩种方法优化该问题。

1. 使用requestAnimationFrame这个API来帮助我们解决这个问题，该方法会在每一帧被调用，通过这个方法的回调参数，我们可以知道每一帧当前还剩余的时间，在时间用完前，让js归还主线程。
2. 使用css中的transform属性，通过该属性实现的动画，不会经过布局和绘制，而是直接运行在合成器线程和栅格线程中，所以不会受到主线程中js执行的影响。更重要的是 transform的动画，由于不需要经过布局绘制样式计算，所以节省了很多运算时间。

### 事件

浏览器通过对不同事件的处理来满足各种交互需求。在浏览器的看来用户的所有手势都是输入，鼠标滚动，悬置，点击等等都是事件。

事件发生时，浏览器进程会发送事件类型及相应的坐标给渲染进程，渲染进程随后找到事件对象并执行所有绑定在其上的相关事件处理函数。

![](https://ae01.alicdn.com/kf/Ucf4a42cc097a4f57b8ebd64e00c163a0W.jpg)

如果页面中没有绑定相关事件，合成器线程可以独立于主线程创建合成帧平滑的处理滚动。如果页面绑定了滚动事件处理器，当页面合成时，合成器线程会标记页面中绑定有事件处理器的区域为非快速滚动区域（non-fast scrollable region），如果存在这个标注，合成器线程会把发生在此处的事件发送给主线程，如果事件不是发生在这些区域，合成器线程则会直接合成新的帧而不用等到主线程的响应。

![](https://ae01.alicdn.com/kf/U57f4f5da0c634feab8c7e0d29a8df8c5C.jpg)

web开发中常用的事件处理模式是事件委托，基于事件冒泡，常常在最顶层绑定事件：

```js
document.body.addEventListener('touchstart', 
event => {
 if (event.target === area) {
 event.preventDefault();
 }
});
```

但是这样整个页面都成了non-fast scrollable region，这意味着即使操作的是页面无绑定事件处理器的区域，每次输入时，合成器线程也需要和主线程通信并等待反馈，流畅的合成器独立处理合成帧的模式就失效了。

![](https://ae01.alicdn.com/kf/U34b6b8c7c535431fb3eef28c5ecb4bc6W.jpg)

为了防止这种情况，可以为事件处理器传递passive: true做为参数，这样写就能让浏览器即监听相关事件，又让合成器线程在等主线程响应前构建新的组合帧。

```js
document.body.addEventListener('touchstart', 
event => {
 if (event.target === area) {
 event.preventDefault()
 }
}, {passive: true}
);
```

不过上述写法可能又会带来另外一个问题，假设某个区域你只想要水平滚动，使用passive: true可以实现平滑滚动，但是垂直方向的滚动可能会先于event.preventDefault()发生，此时可以通过event.cancelable来防止这种情况。

```js
document.body.addEventListener('pointermove', event => {
 if (event.cancelable) {
 event.preventDefault(); // block the native scroll
 /*
 * do what you want the application to do here
 */
 } 
}, {passive: true});
```

也可以使用css属性touch-action来完全消除事件处理器的影响，如：

```css
#area { 
 touch-action: pan-x; 
}
```

**查找到事件对象**

当合成器线程发送输入事件给主线程时，主线程首先会进行命中测试（hit test）来查找对应的事件目标，命中测试会基于渲染过程中生成的绘制记录（paint records）查找事件发生坐标下存在的元素。

![ ](https://ae01.alicdn.com/kf/U46e15d3968894f44a794eeddb7280201l.jpg)

**事件的优化**

一般屏幕的刷新速率为60fps，但是某些事件的触发量会不止这个值，出于优化的目的，Chrome 会合并连续的事件 (如wheel,mousewheel,mousemove,pointermove,touchmove)，并延迟到下一帧渲染时候执行。

而如keydown,keyup,mouseup,mousedown,touchstart和touchend等非连续性事件则会立即被触发。

![](https://ae01.alicdn.com/kf/U02cafd088b2246ca8c819922af24aa8d3.jpg)

合并事件虽然能提示性能，但是如果你的应用是绘画等，则很难绘制一条平滑的曲线了，此时可以使用getCoalescedEvents API来获取组合的事件。示例代码如下：

```js
window.addEventListener('pointermove', event => {
 const events = event.getCoalescedEvents();
 for (let event of events) {
 const x = event.pageX;
 const y = event.pageY;
 // draw a line using x and y coordinates.
 }
});
```

![](https://ae01.alicdn.com/kf/Ued062288b36f42f3b3e4e4385e0c1c5f9.jpg)

文章主要参照:

- https://developers.google.cn/web/updates/2018/09/inside-browser-part1
- https://developers.google.cn/web/updates/2018/09/inside-browser-part2
- https://developers.google.cn/web/updates/2018/09/inside-browser-part3
- https://developers.google.cn/web/updates/2018/09/inside-browser-part4

以及一些网上资料


