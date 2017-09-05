---
layout:     post
title:      "Environment And Tools For iOS Reverse Engineering"
subtitle:   ""
date:       2017-09-05 10:59:59
author:     "Elve Xu"
header-img: "img/in-post/ios-reverse-lis/hacker-bg.jpg"
catalog: true
tags:
    - iOS
    - iOS Reverse Engineering
    - Tools
---

> Code is Not Only Code .


# Config Environment And Tools
## macOS Tools
- class_dump
> Github URL: https://github.com/nygard/class-dump

- ldid & dpkg
> brew install dpkg ldid

- Theos

Install Theos Steps:

> sudo git clone --recursive https://github.com/theos/theos.git /opt/theos
>
> sudo chown $(id -u):$(id -g) /opt/theos
>
> export THEOS="/opt/theos"
>
Check Theos:
>
> /opt/theos/bin/nic.pl NIC 2.0 - New Instance Creator


- Hopper Disassembler v3
> 略

- MachOView(Macho文件浏览器)

> Github URL : https://github.com/gdbinit/MachOView
>
> Download URL : http://sourceforge.net/projects/machoview/

- xcode
> 略

- insert_dylib
> Github URL : https://github.com/Tyilo/insert_dylib.git

- pp助手
> 略

## iOS Devices (Jailbreak)
- cycript
- dumpdecrypted
- debug server
- openssh