---
date: 2022/1/7 18:40
title: Nginx实战之设置反向代理
---



## 反向代理实例一



最终实现的效果就是如下图这样

![](https://picture.xcye.xyz/image-20220107185228644.png)

>  我们直接在浏览器中输入`www.123.com`，那么通过配置nginx反向代理，直接访问Linux中的Tomcat首页，Tomcat的端口为8080



### Linux中启动tomcat

![](https://picture.xcye.xyz/image-20220107191508755.png)



### 修改nginx.conf

![](https://picture.xcye.xyz/image-20220107191553118.png)

我们可以修改默认的server模块，也可以自己增加一个，这里的`server_name`自定义名称，在`location`中，增加`proxy_pass http://127.0.0.1:8080`，这个`proxy_pass`就是配置代理转发，也就是nginx监听的是80端口，但是我们配置了将其转发到`8080`端口

我们也可以将其转发到百度或者是我们自己的博客网页

```conf
location / {
	root   html;
	proxy_pass http://www.baidu.com;
	index  index.html index.htm;
}
```

> 但是这里，我将其转发到`https`时，需要额外设置ssl



## 反向代理实例二

最终实现的效果如下

![](https://picture.xcye.xyz/image-20220107202733674.png)



这个也就是我们同时访问`9001`端口，但是nginx根据访问路径的不同，自动给我们转发到对应的页面



::: tip

这里有一个注意点，比如我们访问`192.168.86.142:9001/edu/a.html`，转发路径为`http://127.0.0.1:8080`(`127.0.0.1`是Linux本机的ip)，那么最终的访问路径为`192.168.86.142:8080/edu/a.html`，访问`192.168.86.142:9001/edu/b.html`，最终的访问路径为`192.168.86.142:8080/edu/b.html`，转发他其实是自动将`/edu/a.html`自动添加到`proxy_pass`的最终路径上

:::



### 初始化tomcat

这里需要用到两个tomcat，一个tomcat的端口为`8080`，另一个tomcat的端口为`8081`



### 修改nginx.conf

```conf
server {
        listen       9001;
        server_name  192.168.86.142:9001;

        location ~ /edu/ {
            proxy_pass http://127.0.0.1:8080;
        }


        location ~ /vod/ {
            proxy_pass http://127.0.0.1:8081;
        }
    }
```



::: tip

`location ~ /deu/`中的`~`表示按照正则表达式进行匹配，这里也就是对于所有的，路径中含有`/edu/`，一定要仔细的写这个匹配的值

:::



![](https://picture.xcye.xyz/image-20220107204140025.png)

![](https://picture.xcye.xyz/image-20220107204213693.png)



## **location 指令说明** 

 该指令用于匹配URL。

![](https://picture.xcye.xyz/image-20220107204543973.png)

1. `=`：用于不含正则表达式的uri前，要求请求字符串与uri严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。

2. `~`：用于表示 uri 包含正则表达式，并且区分大小写。

3. `~*`：用于表示 uri 包含正则表达式，并且不区分大小写。

4. `^~`：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 uri 和请求字符串做匹配。

 注意：如果 uri 包含正则表达式，则必须要有`~`或者`~*`标识。
