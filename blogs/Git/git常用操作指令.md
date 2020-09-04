---
title: git常用操作指令
date: 2020-09-04
tags:
 - git
categories: 
 - git
---
1. 查看远程仓库
``` bash
git remote -v
```
2. 从远程的`origin`仓库的`master`分支下载到本地，并新建一个`dev`分支
``` bash
git fetch origin master:dev
```
3. 查看dev分支与本地当前分支的不同
``` bash
git diff dev
```
4. 将dev分支和当前分支合并
``` bash
git merge dev
```
5. 删除一个dev分支
``` bash
git branch -D dev
```
6. 删除一个远程的分支
``` bash
git push origin --delete dev
```
7. 从**远端dev**分支拉取代码并**新建**一个**本地的dev**分支
``` bash
git fetch origin dev:dev
```
8. 清除未提交缓存区的代码(filename为文件名)
``` bash
git checkout -- filename
```
9. 清除所有
``` bash
git checkout -- .
```