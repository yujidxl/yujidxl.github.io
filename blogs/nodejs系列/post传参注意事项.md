---
title: post传参注意事项
date: 2021-02-19
tags:
  - nodejs系列
categories:
  - nodejs系列
---

1. formdata 类型参数传递
   > 传递 formdata 参数要注意两点
   >
   > - 请求 header 头的设置
   > - 参数需要用 formdata 格式包装：node 中不存在 FromData api,所以需要引入相关 npm 包 `form-data`

```js
const FormData = require("form-data"); // npm的安装包使用方式与浏览器实现的FormData对象相同
const formData = new FormData();
headers: {
  ...formData.getHeaders()
}
```

2. post 参数解析需要借助于 npm 的一个工具包：`koa-bodyparser`
