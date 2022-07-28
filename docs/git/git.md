# 常用命令

保存在暂存去的文件是可以删除的

使用`git rm --cached 文件名`删除的只是暂存区中的文件，在工作区中还存在

![](https://picture.xcye.xyz/image-20210711132329448.png?x-oss-process=style/pictureProcess1)



提交

![](https://picture.xcye.xyz/image-20210711132908923.png?x-oss-process=style/pictureProcess1)



查看日志

![](https://picture.xcye.xyz/image-20210711133114706.png?x-oss-process=style/pictureProcess1)

只要指针指向哪个版本，那么就表示当前最新的提交的版本是哪一个

![](https://picture.xcye.xyz/image-20210711133927455.png?x-oss-process=style/pictureProcess1)

也可以通过下面命令

![](https://picture.xcye.xyz/image-20210711133148294.png?x-oss-process=style/pictureProcess1)



合并冲突

合并冲突，也就是如果我们两个分支，都对内容做了更改，并且提交了，那么使用一个分支，合并另一个分支的时候，就会出现合并冲突，会报下面错误

![](https://picture.xcye.xyz/image-20210711144403878.png?x-oss-process=style/pictureProcess1)

这个时候，就需要使用`vim`命令进入到这个文件中，决定里面内容的去留

![](https://picture.xcye.xyz/image-20210711144259113.png?x-oss-process=style/pictureProcess1)

修改完之后，需要删除`<<<<<HEAD和============和>>>>>>>>hot-fix`

![](https://picture.xcye.xyz/image-20210711144930805.png?x-oss-process=style/pictureProcess1)

修改完之后，需要add，并且需要提交，但是这里提交的时候，不能带上文件名，否则会报错 

![](https://picture.xcye.xyz/image-20210711144616929.png?x-oss-process=style/pictureProcess1)

像这样就合并分支成功了





从远程库拉取到本地库

> git pull 别名 分支



推送

> git push 别名 分支
