---
title: 常用css样式集锦
date: 2020-09-04
tags:
 - css
categories: 
 - css
---
1. 普通适配方案
``` css
@media only screen and (device-width:375px) and (device-height:812px) and (-webkit-device-pixel-ratio:3),
only screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:2),
only screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:3){
  .age-select{
      padding-bottom: 34px !important;
  }
}
```
2. 安全距离适配方案(计算出来的安全高度)
``` css
env(safe-area-inset-bottom)
```
3. css字体粗细一览
- 100 - Thin
- 200 - Extra Light (Ultra Light)
- 300 - Light
- 400 - Normal
- 500 - Medium
- 600 - Semi Bold (Demi Bold)
- 700 - Bold
- 800 - Extra Bold (Ultra Bold)
- 900 - Black (Heavy)
4. 去除滚动条
``` css
::-webkit-scrollbar {
    display: none;
}
```
5. 超出3行文案显示省略号(`注意不能使用text-align: justify;`)
``` css
word-break: break-all;
word-wrap: break-word;
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 3;
-webkit-box-orient: vertical;
```
6. 换行
- 中文换行 
``` css
white-space: normal;
```
- 不换行
``` css
white-space: nowrap;
```
- 英文或数字的换行
``` css
 word-break: break-all;  // 单个字母换行(不保证单词的完整性)以字母形式换行
 word-break: break-word; // 单个单词的换行(保证单词的完整性)以单词形式换行
```