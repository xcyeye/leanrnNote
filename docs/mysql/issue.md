---
date: 2022/3/10 10:46
---



# mysql问题集合



## 允许远程链接



```mysql
# 错误
ERROR 1130: Host 'xxx' is not allowed to connect to this MySQL server
```



解决

```mysql
use mysql;

update user set host = '%' where user = 'root';

# 一定要执行下面这步
flush privileges; 
```



## mac启动mysql服务

```sh
sudo /usr/local/mysql/support-files/mysql.server start
```

> 非常感谢[mac怎么启动mysql服务](https://www.yisu.com/ask/864.html#:~:text=在mac中启动mysql服务的方法 1.首先，在mac桌面侧边栏中点击“应用程序”，选择“实用工具”选项； 2.在实用工具中，双击打开“终端”应用程序； 3.进入到终端应用程序后，使用以下命令启动mysql服务；,sudo %2Fusr%2F local %2Fmysql%2Fsupport-files%2Fmysql.server start)
