---
layout:     post
title:      Linux系统命令-htop
subtitle:   htop命令
date:       2021-12-16
author:     dm
header-img: img/post-bg-mma-3.jpg
catalog: true
tags:
    - Linux





---

## top命令升级版本htop

- htop的介绍
  `htop`是Linux系统中的一个互动的进程查看器,与Linux传统的top比较的话,`htop`更`人性化`并且还`支持鼠标`操作!
- htop的优势

> (1) 在htop中，可以垂直和水平滚动列表，查看所有进程和完整的命令行。
> (2) 在top中，您按下的每个未分配的键都有延迟(尤其是当多键转义序列意外触发时)。
> (3) htop启动得更快(top似乎在显示任何东西之前会收集一段时间的数据)。
> (4) 在htop中，您不需要输入进程号来终止进程，而在top中，您需要这样做。
> (5) 在htop中，您不需要输入进程编号或优先级值来重新分配进程，而在top中，您需要这样做。
> (6) 在htop中，您可以同时杀死多个进程。
> (7) top更老，因此更容易测试。

- htop的安装
  [htop:项目地址](https://github.com/hishamhm/htop)
  
  [htop官网地址](http://htop.sourceforge.net/)
  
  [Linux](https://so.csdn.net/so/search?from=pc_blog_highlight&q=Linux)
  
  [top 命令的用法详细详解](https://www.cnblogs.com/zhoug2020/p/6336453.html)
  
  [htop 使用详解](https://www.cnblogs.com/programmer-tlh/p/11726016.html)

> 可以通过`源码包编译`安装
> 也可以`配置epel源`后`yum安装`

这里使用CentOS7.X系统版本为例,使用`yum`下载安装:

```bash
#安装epel源
> yum install epel-release
#安装htop
> yum install -y htop
#安装完毕后命令行输入
> htop
```

![htop展示区域](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E5%B1%95%E7%A4%BA%E5%8C%BA%E5%9F%9F.png)

说明:

> 从上面的截图中可以看到,`htop`命令输出总共分成了五个展示区:
> (1)CPU状态区域
> (2)整体状态区域
> (3)内存状态区域
> (4)进程状态区域
> (5)管理控制区域

## htop的众多输出信息的详解

htop通过进度条展示每个CPU逻辑核心的使用百分比,并使用不同的颜色进行区分:

- CPU usage bar

![htop之CPU使用情况](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8BCPU%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5.png)

> 该行主要显示CPU使用情况,htop还为将不同颜色来区分是使用情况:
> (1)蓝色的表示low-prority(低优先级)使用
> (2)绿色的表示normal(标准)使用情况
> (3)红色的表示kernel(内核)使用情况
> (4)青色的表示virtuality(虚拟性)使用情况

- Memory bar

![htop之内存使用情况](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8B%E5%86%85%E5%AD%98%E4%BD%BF%E7%94%A8%E6%83%85%E5%86%B5.png)

> 该行主要表示内存使用情况，同样的htop使用了不同颜色来区分是使用情况:
> (1)绿色的表示已经使用内存情况
> (2)蓝色的表示用于缓冲的内存使用情况
> (3)黄色的表示用于缓存的内存使用情况

- Swap bar

> 该行主要显示交换分区使用情况，当你发现你的`交换分区(swap)已经派上用场`的时候，说明你的物理**内存已经不足**，需要考虑增加内存了。

- 整体状态区域

![htop之整体状态区域](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8B%E6%95%B4%E4%BD%93%E7%8A%B6%E6%80%81%E5%8C%BA%E5%9F%9F.png)

> (1)Tasks显示进程总数,当前运行的进程数
> (2)Load average显示的是系统的1分钟,5分钟,10分钟的平均负载情况
> (3)Uptime显示系统运行了多长时间

- 进程状态区域

> PID：表示进程号,是非零正整数
> USER：发起该进程的用户名
> PRI：进程优先级
> NI：(nice)进程的优先级别数值
> VIRT：进程占用的虚拟内存
> RES：进程占用的物理内存
> SHR：进程使用的共享内存
> S：进程的运行状况
>
> (1) R 表示正在运行
> (2) S 表示休眠
> (3) Z 表示僵死状态
> (4) N 表示该进程优先值是负数
>
> CPU%：进程占用的CPU使用率
> MEM%：此进程占用的物理内存和总内存的百分比
> TIME%：启动进程后占用CPU的累计时长
> Command：进程启动的启动命令名称即路径

- 管理控制台

> F1：查看htop说明
> F2：htop设定
> F3：搜索进程
> F4：进程过滤器
> F5：显示属性结构
> F6：折叠或展开(新版本里的),或选择排序方式(旧版本里的)
> F7：减少nice值,提高进程优先级
> F8：增加nice值,降低进程优先级
> F9：可对进程传递信号
> F10：退出

**Setup 选项下的：**

> **1. Meters：**设定顶端的显示信息，分为左右两侧，Left column 表示左侧的显示的信息，Right column表示右侧显示的信息，如果要新加选项，可以选择Available meters添加，F5新增到上方左侧，F6新增到上方右侧。Left column和Right column下面的选项，可以选定信息的显示方式，有LED、Bar(进度条)、Text(文本模式)，可以根据个人喜好进行设置
>
> **2. Display options：**选择要显示的内容，按空格 x 表示显示，选择完后，按F10保存
>
> **3. Colors：**设定界面以什么颜色来显示，个人认为用处不大，各人喜好不同
> **4. Colums：**作用是增加或取消要显示的各项内容，选择后F7(向上移动)、F8(向下移动)、F9(取消显示、F10(保存更改))此处增加了PPID、PGRP，根据各人需求，显示那些信息。

![htop之Setup选项](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8BSetup%E9%80%89%E9%A1%B9.png)

> Meters 页面设定了顶端的一些信息显示，顶端的显示又分为左右两侧，到底能显示些什么可以在最右侧那栏新增，要新增到上方左侧（F5）或是右侧（F6）都可以，这就是个人设定的范围了。这里多加了一个时钟。

![htop之Setup](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8BSetup.png)

> 我们也可以自定义进程区域中的显示内容：

![htop之Setup自定义](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8BSetup%E8%87%AA%E5%AE%9A%E4%B9%89.png)

> **搜索进程：**鼠标点击Search 或者按下F3 或者输入"/"， 输入进程名进行搜索，例如搜索ssh
> **过滤器：**按下F4，进入过滤器，相当于关键字搜索，不区分大小写，例如过滤dev。
> **search 和 filter 的区别 ：**search 会将光标定位到符合条件的进程上，通过 F3 键进行逐个查找；而 filter 会直接将符合条件的进程筛选出来。search 和 filter 都使用 ESC 键来取消功能。
> **显示树形结构：**输入 "t" 或 按下 F5，显示树形结构，意思跟 pstree 差不多，能看到所有程序树状执行的结构，这对于系统管理来说相当方便，理清程序是如何产生的，当然树状结构的浏览也可以依照其他数据来排序。退出树状视图模式，请再一次按下 F5 键。 
> **选择排序方式：**按下 F6 就可以选择依照什么来排序，最常排序的内容就是 cpu 和 memory 吧！
> **F7、F8分别对应 nice- 和 nice+**：F7 表示减小 nice值(增大优先级)，F8增大nice值(减小优先级)，选择某一进程，按F7或F8来增大或减小nice值，nice值范围为-20-19
> **F9 对应 kill 给进程发信号**，选好信号回车就OK了（ **F9：杀掉指定进程**）。选择某一进程按 F9 即可杀死此进程，如下图所示：窗口的左边部分列出的是所有可用的信号，右边部分列出的是进程。只要选中信号，并选择一个进程，然后按下 enter 键，选中的信号就会发送到此进程。

![htop之Setup2](https://raw.githubusercontents.com/DongMing0103/MarkdownCloudImage/master/data/htop%E4%B9%8BSetup2.png)

> **F10：**退出htop。
>
> **空格键**：用于标记选中的进程，用于实现对多个进程同时操作；要标注某个进程条目，需要做的就是选中此条目，然后按下空格键。
>
> **显示某个用户的进程：**在左侧选择用户：输入"u"，在左侧选择用户



- 命令行选项（COMMAND-LINE OPTIONS）

| -C --no-color        | 使用一个单色的配色方案（设置界面为无颜色）                   |
| -------------------- | ------------------------------------------------------------ |
| -d --delay=DELAY     | 设置延迟更新时间，单位秒（设置刷新时间，单位为秒） 例如，htop -d 100 命令会使输出在1秒后才会刷新（参数 -d 的单位是10微秒）。 |
| -h --help            | 显示htop 命令帮助信息                                        |
| -u --user=USERNAME   | 只显示一个给定的用户的过程（显示指定用户的进程） 例如，htop -u himanshu 命令会只显示出用户名为 himanshu 的相关进程。 |
| -p --pid=PID,PID…    | 只显示给定的PIDs                                             |
| -s --sort-key COLUMN | 依此列来排序（以指定的列排序）。 例如，htop -s PID 命令会按 PID 列的大小排序来显示。 |
| -v –version          | 显示版本信息                                                 |
