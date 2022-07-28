---
date: 2022/1/3 17:29
title: 看鸟哥的私房菜这本书的学习笔记
tag: [linux]
---

## 终端登录Linux系统

![](https://picture.xcye.xyz/image-20220103173211054.png)

终端登录Linux系统，其界面也就是上图这样，我们看到当前的发行版和Linux内核版本

这里我们可以在这个界面上，添加一些新的东西，需要在`etc/issue`这个文件中，进行修改

![](https://picture.xcye.xyz/image-20220103173502328.png)

如果我们修改了这个文件，我们可以注销，就可以看到效果了，使用`exit`注销



## 注销

注销使用`exit`，注销并不是关机，其只是当前用户离开系统，只表示当前用户所执行的任务结束了，但是还有很多的其他用户的任务在执行



## 命令行模式下，命令执行

如果我们输入的一行命令太长的话，我们可以通过`\`反斜杠来转义回车键，输入`\`后，我们需要按回车，紧接着继续输入命令

![](https://picture.xcye.xyz/image-20220103174759046.png)





## 修改当前输出语言

下面的操作只能修改当前用户未注销前的输出语言，如果用户使用`exit`注销之后，那么修改过的操作都不会生效，默认情况下，输出的语言，会使用当前Linux系统的语言作为输出语言

```sh
[root@qsyyke ~]# locale
LANG=zh_CN.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=
```

![](https://picture.xcye.xyz/image-20220103183236260.png)

通过`locale`指令我们可以看到与每一个相关的语言

### 修改为英文

通过`LANG=xxx`可以修改输出信息语言

```sh
[root@qsyyke ~]# LANG=en_US.utf-8 or LANG=en_US.utf8
[root@qsyyke ~]# date
Mon Jan  3 18:53:58 CST 2022
```

::: warning 

`LANG`只与输出信息相关，如果需要修改其他的信息，那么需要修改`LC_ALL`

```sh
export LC_ALL=en_US.utf8
```

:::



当使用`exit`注销之后，那么我们再次登录进去，这些修改都无效

```sh
[root@qsyyke ~]# exit
logout

Connection closed.

Disconnected from remote host(本地centos-root) at 19:23:16.

Type `help' to learn how to use Xshell prompt.
[C:\~]$ 

..............

Last login: Mon Jan  3 18:26:41 2022 from 192.168.86.1
[root@qsyyke ~]# date
2022年 01月 03日 星期一 19:23:22 CST
```



## man指令

通常man page的数据时存放在`/etc/share/man`文件夹中的，我们可以通过修改配置文件，修改他的man page查找路径来改善这个目录的问题，需要修改`/etc/man_db.conf`，有些发型版本或者版本可能不同



## 正确关机方式

我们可以使用`who`这个指令，看一下有哪些用户当前在线，这样可以避免我们直接关机，对其他的用户造成影响		
