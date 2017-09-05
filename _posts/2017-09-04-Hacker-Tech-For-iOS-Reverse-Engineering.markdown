---
layout:     post
title:      "0x00. Hacker Tech For iOS Reverse Engineering"
subtitle:   ""
date:       2017-09-04 10:59:59
author:     "Elve Xu"
header-img: "img/in-post/ios-reverse-lis/hacker-bg.jpg"
catalog: true
tags:
    - iOS
    - iOS Reverse Engineering
---

> Code is Not Only Code .


# Mobilesubstrate
> 下面的图展示的就是oc届著名的method swizzling技术，他就是iOS的注入原理，类似于windows的钩子

![image](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/ios-reverse-lis/EC2A4A79-44C7-46A0-9F13-674436E9D060.png)

> Mobilesubstrate为了方便tweak开发，提供了三个重要的模块：

- MobileHooker 就是用来做上面所说的这件事的，它定义一系列的宏和函数，底层调用objc－runtime和fishhook来替换系统或者目标应用的函数
- MobileLoader 用来在目标程序启动时根据规则把指定目录的第三方的动态库加载进去，第三方的动态库也就是我们写的破解程序，他的原理下面会简单讲解一下
- Safe mode 类似于windows的安全模式，比如我们写的一些系统级的hook代码发生crash时，mobilesubstrate会自动进入安全模式，安全模式下，会禁用所有的第三方动态库

> 参考链接: http://nshipster.com/method-swizzling/

# iOS App注入原理
> 从下面的图来看，Mach-O文件的数据主体可分为三大部分，分别是头部（Header）、加载命令（Loadcommands）、和最终的数据（Data）。
> mobileloader会在目标程序启动时，会根据指定的规则检查指定目录是否存在第三方库，如果有，则会通过修改二进制的loadCommands，来把自己注入进所有的app当中，然后加载第三方库。

![image](https://raw.githubusercontent.com/misselvexu/misselvexu.github.io/master/img/in-post/ios-reverse-lis/D72609C1-AB49-495D-AAB4-6CB1E92F9F2A.png)

