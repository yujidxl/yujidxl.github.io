---
title: git换行符处理
date: 2020-09-04
tags:
 - git
categories: 
 - git
---
**windows**: CR回车 LF换行 CRLF \r\n
<br />**linux|unix**: LF \n
<br />**macos**: CR \r
> `git同步文件解决办法：`
打开命令行，进行设置，如果你是在Windows下开发，建议设置autocrlf为true
<br />补充：如果你文件编码是UTF8并且包含中文文字，那还是把autocrlf设置为false，并且把所有文件转换为Linux编码（即LF\n），
开启safecrlf检查。

## AutoCRLF的可配置实例
1. 提交时转换为`LF`，检出时转换为`CRLF`
``` bash
git config --global core.autocrlf true   
```

2. 提交时转换为`LF`，检出时`不转换`
``` bash
git config --global core.autocrlf input   
```

3. 提交检出`均不转换`
``` bash
git config --global core.autocrlf false
```

## SafeCRLF的可配置实例
1. 拒绝提交包含混合换行符的文件
``` bash
git config --global core.safecrlf true   
```
2. 允许提交包含混合换行符的文件
``` bash
git config --global core.safecrlf false   
```

3. 提交包含混合换行符的文件时给出警告
``` bash
git config --global core.safecrlf warn
```

:tada: