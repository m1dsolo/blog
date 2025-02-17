---
title: '在 dwm 中防止特定窗口被 swallow 的方法'
date: '2025-02-17'
tags: ['dwm']
draft: false
keywords: [
    'dwm',
    'swallow'
]
---

## 1. 前言

dwm 是 X11 下的平铺式窗口管理器，它是用 C 语言编写的，非常轻量级，全部代码也才不到 2k 行。
它提供了一些基本的窗口管理功能，对于额外的功能，可以通过修改源码来实现。
目前 [suckless 官网](https://dwm.suckless.org/patches/) 上提供了非常多的补丁，可以用来扩展 dwm 的功能。

其中一个很重要的补丁就是 [swallow](https://dwm.suckless.org/patches/swallow/) ，
它可以让 dwm 在启动一个新的应用程序时，将其嵌入到一个已有的窗口中，而不是启动一个新的窗口。
这样可以大幅度减少多余窗口的数量。

![swallow](https://m1dsolo.xyz/images/dwm-noswallow/swallow.gif)

详细信息可以参考 [Luke Smith 的视频](https://www.youtube.com/watch?v=92uo5OBOKfY&t=327s) 。

## 2. 需求分析

swallow 固然很好，但是有时候我们并不希望所有的窗口都 swallow 终端窗口，
比如有时候我在开发游戏过程中，我希望可以看到终端窗口的输出来进行 debug ，这时 swallow 补丁就无法满足我的需求了。

我尝试过其他方式，比如使用 [dynamicswallow patch](https://dwm.suckless.org/patches/dynamicswallow/) ，
但是这个 patch 过于重量级，提供了太多不必要的功能。
所以我决定在 swallow patch 的基础上进行修改，实现只有特定窗口才会被 swallow 的功能。

![noswallow](https://m1dsolo.xyz/images/dwm-noswallow/noswallow.gif)

上面演示了`noswallow`的效果：
通过在执行命令前加上`noswallow`脚本，
比如`noswallow ./your_app`，就可以防止`your_app`被 swallow 。

## 3. 实现原理

实现过程非常简单：

因为我们需要进行进程间通信（终端和 dwm 之间），
所以我们选择最简单的进程间通信方式：文件。

在 dwm 的`dwm.c`中的`swallow`函数开始处，我们添加一个`if`语句：
```c
void
swallow(Client *p, Client *c)
{
    if (c->noswallow || c->isterminal)
        return;
    if (c->noswallow && !swallowfloating && c->isfloating)
        return;
    // add this to your code
    if (!access("/tmp/noswallow", F_OK))
        return;
```

`access("/tmp/noswallow", F_OK)`会检查`/tmp/noswallow`是否存在，如果存在则返回 0 ，否则返回-1 。

这样我们就可以通过创建或删除`/tmp/noswallow`文件来控制是否 swallow 。

注意需要`sudo make clean install`重新编译安装 dwm 。

接下来编写一个简单的脚本`noswallow`：
```bash
#!/bin/sh

touch /tmp/noswallow

"$@"  # run your command with arguments

rm /tmp/noswallow
```

这样在执行命令前加上`noswallow`脚本，就可以防止该窗口被 swallow 了！（记得将`noswallow`加到环境变量中。）

具体可以参考我的 [commit](https://github.com/m1dsolo/dotfiles/commit/2ce537186b1d22201fd052f4891df09060468efd) 。
