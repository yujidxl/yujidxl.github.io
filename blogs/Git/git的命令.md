---
title: git的配置
date: 2020-09-04
tags:
 - git
categories: 
 - git
---
## 查看
1. 查看系统配置
``` bash
git config --system --list
```
2. 查看当前用户配置
``` bash
git config --global --list 
```
3. 查看当前仓库配置
``` bash
git config --local --list
```
## 设置 
``` bash
git config --global 参数名 参数值
```