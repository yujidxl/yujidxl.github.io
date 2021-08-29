---
title: preload & prefetch
date: 2020-09-04
tags:
 - 前端性能
categories: 
 - 基础知识
---
## preload & prefetch
1. preload: `提前加载`
> preload 顾名思义就是一种预加载的方式，它通过声明向浏览器声明一个需要提交加载的资源，当资源真正被使用的时候立即执行，就无需等待网络的消耗。
``` html
<!-- 使用 link 标签静态标记需要预加载的资源 -->
<link rel="preload" href="/path/to/style.css" as="style">

<!-- 或使用脚本动态创建一个 link 标签后插入到 head 头部 -->
<script>
const link = document.createElement('link');
link.rel = 'preload';
link.as = 'style';
link.href = '/path/to/style.css';
document.head.appendChild(link);
</script>
```
2. prefetch: `预判加载`
> prefetch 跟 preload 不同，它的作用是告诉浏览器未来可能会使用到的某个资源，浏览器就会在闲时去加载对应的资源，若能预测到用户的行为，比如懒加载，点击到其它页面等则相当于提前预加载了需要的资源。它的用法跟 preload 是一样的：
``` html
<!-- link 模式 -->
<link rel="prefetch" href="/path/to/style.css" as="style">

<!-- HTTP 响应头模式 -->
Link: <https://example.com/other/styles.css>; rel=prefetch; as=style
```

## 细节注意点
> 当一个资源被 preload 或者 prefetch 获取后，它将被放在内存缓存中等待被使用，如果资源位存在有效的缓存极致（如 cache-control 或 max-age），它将被存储在 HTTP 缓存中可以被不同页面所使用。

正确使用 preload/prefetch 不会造成二次下载，也就说：当页面上使用到这个资源时候 preload 资源还没下载完，这时候不会造成二次下载，会等待第一次下载并执行脚本。

对于 preload 来说，一旦页面关闭了，它就会立即停止 preload 获取资源，而对于 prefetch 资源，即使页面关闭，prefetch 发起的请求仍会进行不会中断。
