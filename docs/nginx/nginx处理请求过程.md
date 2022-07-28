---
date: 2022/1/24 12:31
---

# NGINX处理请求过程

在开始之前，需要了解`server`块中的`server_name`是定义服务器名称的，这里可以是ip或者域名，还可以是空字符串，如果你想要某个域名被这个服务器所处理，那么就需要将`server_name`设置为你的域名



```conf
server {
    listen      80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      80;
    server_name example.com www.example.com;
    ...
}
```

nginx会根据请求头中的`host`来决定此请求应该路由到哪个服务器(`根据server块中的server_name判断`，ip地址也是host)，如果这个host没有和之匹配的`server_name`或者这个请求头中没有包含host，那么nginx将会将此请求路由到默认的服务器来进行处理

- 默认服务器

  1. 如果某个server的`server_name`是字符串，那么这个server就是一个默认服务器

  2. 还有也可以通过`listen`指定

     > ```conf
     > server {
     >     listen      80 default_server;
     >     server_name example.net www.example.net;
     >     ...
     > }
     > ```



## 基于server_name和基于ip的混合虚拟服务器

- 基于ip？

  > 这是我的理解，基于ip就是listen是ip和端口

所以标题也就是说，如果某个server块中listen是ip，这种和server_name混合的情况

```conf
server {
    listen      192.168.1.1:80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      192.168.1.1:80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      192.168.1.2:80;
    server_name example.com www.example.com;
    ...
}
```

对于这种server_name和listen是ip的情况，其处理时

1. 先测试此listen部分的ip和端口，看看是否能够正常使用

2. 如果ip和端口正常，那么nginx便会根据ip和端口去找和host匹配的`server_name`

   > 根据ip和端口？
   >
   > 我的理解是：不同的host他们指向不同的端口

3. 如果没有和host匹配的`server_name`，那么此请求将会被默认的server所处理



比如这里有一个请求`www.example.com`，此请求会被`192.168.1.1:80`所接收，所以会被第三个服务器所处理



我这里有一个测试配置

```conf
server {
        listen 192.168.1.1:8085;
        location /bb {
                root /data/www;
        }
    }

```

> 没有开放8085端口，所以当我保存配置之后，执行`nginx -t`，那么会得到下面错误，在error.log和access.log中，还有运行`nginx -t`
>
> ```
> error.log和access.log中得到的内容一样
> 2022/01/24 14:20:32 [emerg] 48860#0: bind() to 192.168.1.1:8085 failed (99: Cannot assign requested address)
> ```
>
> ```bin
> #运行nginx -t得到的结果
> [root@qsyyke-host conf]# ../sbin/nginx -t
> nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
> nginx: [emerg] bind() to 192.168.1.1:8085 failed (99: Cannot assign requested address)
> nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
> ```
>
> 端口绑定不成功

