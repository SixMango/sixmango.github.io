---
layout: post
title: 关于前端缓存优化
date: 2020-12-30 14:42:40
tags:
- cache
- http
categories:
- 前端
---

在浏览器通过DNS域名解析获取到IP地址后，在发送请求时会先进行缓存检测。

> 缓存位置：
> 
> - Memory Cache：内存缓存
> - Disk Cache：硬盘缓存

打开网页：查找disk cache中是否有匹配，有则使用

普通刷新(F5)：网页没有关闭，优先使用memory cache，其次使用disk cache

强制刷新(Ctrl + F5)：不使用缓存，请求头设置Cache-control:no-cache

## 强缓存 Expires / Cache-Control

浏览器对于强缓存的处理，根据第一次请求资源时返回的响应头来确定的

- Expires：缓存过期时间,用来指定资源到期的时间(HTTP/1.0)
- Cache-Control：cache-control:max-age=2592000第一次拿到资源后的259200秒内(30天),再次发送请求,读取缓存中的信息(HTTP/1.1)
- 两者同时存在的话, Cache-Control优先级高于Expires

> HTML页面一般不做强缓存，以便获取的html数据最新

### 缺点

如果服务器文件更新了，但本地存在强缓存，这种情况是拿不到最新内容的，可以通过以下方法解决

1）服务器更新资源后，让资源名称和之前不一样，例如：

```
index. dads 3232. js
index. fsdfsddvd js
```

可以使用webpack打包工具中设置hash name

2）当文件更新后,我们在htm倒入的时候,设置个后缀(时间戳)

```
<script src="index. js?a=312312321">
<script src="index js?a=898989888">
```

3）使用协商缓存

### 优点

强缓存要比协商缓存性能高（不必总是向服务器发请求）

## 协商缓存 Last-Modified / ETag

协商缓存在强缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否需要更新

- Last-Modified：资源文件最后更新的时间(HTTP/1.0)
- ETag：记录资源的一个标识，每一次资源更新都会重新生成一个Etag(HTTP/1.1)
- 两者同时存在的话, ETag优先级高于Last-Modified

浏览器将信息和标识缓存到本地，再次发起请求时在请求头中携带

If-Modifhed-Since：Last-Modified

If-None-Match：ETag

**强缓存和协商缓存主要用作静态资源文件，一般不经常更新**

## 数据缓存

使用sessionStorage/localStorage或者vuex/redux等，将通过Ajax获取到的数据进行存储。

## DNS缓存

在做DNS解析时会先查找本地是否有缓存。

> 每一次DNS解析时间预计在20~120毫秒
> 
> - 减少DNS请求次数
> - DNS预获取( DNS Prefetch)

由于要保持高可用，高并发。提供不同数据的服务会部署在不同的服务器上，所以也会导致DNS解析消耗的时间增加，这种情况可以使用DNS预获取。

```html
<link rel="dns-prefetch" href="//static.36obuyimg.com"/>
<link rel="dns-prefetch" href="//d3.cn"/>
<link rel="dns-prefetch" href="//djd.com"/>
```
