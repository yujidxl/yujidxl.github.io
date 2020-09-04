---
title: vue3.0响应式原理
date: 2020-09-04
tags:
 - js
categories: 
 - js
---
## 时间史
在`vue2.0`中使用的`Object.defineProperty`或者`Object.defineProperties`，但是到`3.0`时vue使用`proxy`，此api没有shame方案，全看浏览器支持，所以如果你需要适配一些低版本的浏览器就需要三思而后行了。
``` js
const obj = { name: "krry", age: 24, others: { mobile: "mi10", watch: "mi4" } };
const p = new Proxy(obj, {
  get(target, key, receiver) {
    console.log("查看的属性为：" + key);
    return Reflect.get(...arguments);
  },
  set(target, key, value, receiver) {
    console.log("设置的属性为：" + key);
    console.log("新的属性：" + key, "值为：" + value);
   return Reflect.set(...arguments);
  },
});
```