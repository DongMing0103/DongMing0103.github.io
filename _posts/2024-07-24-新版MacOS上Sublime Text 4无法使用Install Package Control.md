---
layout:     post
title:      新版MacOS上Sublime Text 4无法使用Install Package Control
subtitle:   
date:       2024-07-24
author:     dm
header-img: img/post-bg-BJJ.jpg
catalog: true
tags:
    - Sublime





---



## 新版MacOS上Sublime Text 4无法使用Install Package Control

新版`MacOS`上`Sublime Text 4`无法使用`Install Package Control`或安装`Install Package Contro`找不到`Install Package`选项的解决办法：

> 最新升级了MacOS到Ventura和Sonoma,发现安装Sublime Text4之后，无法使用Install Package Control，即使安装了Install Package Control也无法正常使用Install Pakcage。确实要命啊。

### 解决方法：

**下载PackageControl：**[PackageControl](https://github.com/wbond/package_control/releases)

之后得到一个`Package.Control.sublime.package`文件，将文件重命名为<font color = 'red'>Package Control.sublime-package</font>
随后将这个文件放置到插件安装目录的`（Installed Packages）`目录下，路径为：***/Users/用户名/Library/Application Support/Sublime Text/Installed Packages***

这里有个坑，<font color = 'red'>不是Browser Package浏览插件操作打开的目录</font> ，而是这个目录的上级目录。浏览插件打开的是User下的，而`Installed Packages目录和User同级`。**需要将插件放到Installed Packages目录下。**

放置好之后，**重启Sublime**就可以正常使用Sublime Text了

[CSDN图片](https://blog.csdn.net/qq_42687675/article/details/132811797)
