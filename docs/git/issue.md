# git使用遇到问题

## 报错443

解决方法可以查看https://juejin.cn/post/6844904193170341896

设置完成之后，还需要刷新一下cdn

```sh
ipconfig /displaydns
ipconfig /flushdns
```

## git clone指定分支

```git
git clone -b 分支 地址
```





## push出现 Updates were rejected because the tip of your current branch is behind

如果将本地库push到远程库，报下面错误

```git
error: failed to push some refs to 'https://github.com/qsyyke/vuepress-theme-ccds.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

出现这个问题是因为，在push之前，需要将本地库的变化，连接到远程库

解决

```git
#先pull
git pull main master

#在push
git push main master
```





## 获取最新版本接口

```
https://api.github.com/repos/qsyyke/vuepress-theme-ccds/releases/latest
```
