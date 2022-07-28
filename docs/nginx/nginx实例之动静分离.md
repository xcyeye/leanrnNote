---
date: 2022/1/7 22:12
title: Nginx实现动静分离
---

其实动静分离，也就是我们可以将动态资源(`jsp,php`)等和静态资源(`html,css,js`)等进行分开，比如动态资源放在tomcat，静态资源我们可以放在Linux根目录下的static(`未存在，需要自己创建`)中，我们通过nginx进行配置，可以通过ip:端口去访问`/static`目录下的静态文件



Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用`Nginx`处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种，一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案；

另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。

通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。

具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可，所以不会产生额外的流量。

此种方法非常适合不经常变动的资源。（如果经常更新的文件，不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送

一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码304，如果有修改，则直接从服务器重新下载，返回状态码 200。



## 准备资源

```sh
[root@qsyyke www]# ls
a.html
[root@qsyyke www]# pwd
/static/www

[root@qsyyke www]# cd /static/image/
[root@qsyyke image]# ls
10.jpg  13.jpg  14.jpg  1.jpg
[root@qsyyke image]# pwd
/static/image
```



## 修改nginx.conf

```conf
server {
        listen       80;
        server_name  192.168.86.142-tomcat;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;


        location /www/ {
           root  /static/;
           index  index.html index.htm;
        }


        location /image/ {
           root  /static/;
           autoindex  on;
        }

}
```

> 这里的`root`就是这些静态资源的根目录在哪里，当nginx服务器去寻找这些静态文件时，是通过`{root}/{/www/ or /image/}`去寻找资源的

访问

- http://192.168.86.142/image/13.jpg
- http://192.168.86.142/www/a.html

都成功访问到资源



