---
date: 2022/1/24 12:20 
tag: [nginx]
---

# nginx中的命令行参数

我们常用的参数有`-t -s`等

但是除此以外，还有其他的命令行参数

- -c file

  > 使用某个文件，替换默认配置文件
  >
  > ```sh
  > [root@qsyyke-host conf]# ../sbin/nginx -c /usr/local/nginx/conf/nginx_fuben2.conf
  > ```

- `-p prefix` 

  > 设置nginx路径前缀，即保存服务器文件的目录（默认值为`/usr/local/nginx`）

- `-q` 

  > 在配置测试期间抑制非错误消息。

- `-t` 

  > 测试配置文件：nginx检查配置的语法是否正确，然后尝试打开配置中引用的文件。

- `-T` 

  > 与 相同`-t`，但另外将配置文件转储到标准输出 (1.9.2)。

- 
