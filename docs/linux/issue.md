---
date: 2022/3/1 22:56
---

# Linux的一些问题总结

这里记录了在使用Linux的过程中的一些问题总结



## 使用vim打开文件，中文乱码问题

```sh
vim /etc/vimrc
```

然后添加下面内容，然后保存就行

```sh
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
```

感谢[linux -- 解决配置vim中文乱码的问题 - Be-myself - 博客园 (cnblogs.com)](https://www.cnblogs.com/gengyufei/p/13354637.html)提供的解决方法





## 在centos上安装MySQL

[如何在 CentOS 8 上安装 MySQL - 云+社区 - 腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1626795)





## 在centos上安装php环境

[(73条消息) linux搭建初始php环境（极简！）_954L-CSDN博客_linux安装php环境](https://blog.csdn.net/wkh___/article/details/83183621)
