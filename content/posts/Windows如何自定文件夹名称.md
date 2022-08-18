---
title: Windows如何自定文件夹名称
date: 2017-09-17 13:30:47
tags:
---

比如，桌面文件夹，其实真实的路径是
```
C:\Users\用户名\Desktop
```
而不是桌面。

<!--more-->

方法很简单。
假设要创建一个文件夹，名称叫做“代码”，路径是“code” 首先新建一个文件夹，名为code。
在文件夹下新建文件：desktop.ini 文件内容为：

```bat
[.ShellClassInfo]
LocalizedResourceName=代码
```

保存。 最后，在当前目录中打开cmd，执行命令

```bat
attrib +s .
```

为当前文件夹添加系统属性。
**注意：** 文件夹的真实路径必须是不带空格的英文。