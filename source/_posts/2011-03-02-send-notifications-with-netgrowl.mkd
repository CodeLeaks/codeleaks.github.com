--- 
categories: 
  - Python
comments: true
layout: post
published: true
status: publish
tags: []
title: "利用 netgrowl 向 Windows / Mac OS X 发送消息"
type: post
---
我平时用的系统是 Windows 7 和 Mac OS X，实验室项目一般都是 ssh 远登到 Ubuntu 和 Linux 上开发的。有时碰到内核和虚拟机等项目编译比较耗时，编译开始后要时不时的看一下编译任务是否完成，或者有没有中途出错，这时候如果有个通知系统就比较方便了。

Google 了一把找到了 <a title="netgrowl" href="http://the.taoofmac.com/space/projects/netgrowl" target="_blank">netgrowl</a> 这个好东东，它是一个开源的 Python 模块，实现了 Growl 协议，可以向 Mac 或 Windows 上的 Growl 服务发送通知。使用也非常方便，先用 GrowlRegistrationPacket 函数注册一个应用，接着就可以用 GrowlNotificationPacket 发送通知了：

<u>notify.py</u>


``` 
#!/usr/bin/python

from netgrowl import *
import sys

title = "Notification from Ubuntu"
desc = ""
if len(sys.argv) > 2:
    title = sys.argv[1]
    desc = sys.argv[2]

addr = ("10.131.251.101", GROWL_UDP_PORT)
s = socket(AF_INET,SOCK_DGRAM)
p = GrowlRegistrationPacket(application="Ubuntu", password="i")
p.addNotification("Ubuntu Notifications", enabled=True)
s.sendto(p.payload(), addr)
p = GrowlNotificationPacket(application="Ubuntu",
    notification="Ubuntu Notifications", title=title,
    description=desc, priority=1,
    sticky=True, password="i")
s.sendto(p.payload(),addr)
s.close()
```


这里的 addr 是接收方的地址，GrowlRegistrationPacket 和 GrowlNotificationPacket 中需要指定 Growl 远程服务的密码。

然后是一个简化 notify.py 调用的 shell 脚本：

<u>growl.sh</u>


``` 
#!/bin/bash

cmd=$@
$cmd
python ~/bin/notify.py Done "$cmd under $PWD is finished"
```


把 growl.sh 加入到 PATH 中，之后只要运行 <u>growl.sh make all</u> 就能运行 make all 命令 ，并且在执行完成后向 Growl 客户端发送消息了。P.S. Growl for Windows 可以在<a title="这里" href="http://www.growlforwindows.com/" target="_blank">这里</a>找到。
