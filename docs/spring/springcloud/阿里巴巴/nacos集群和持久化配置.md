---
date: 2022/2/18 23:28
title: nacos集群和持久化配置
---



我们不仅可以在服务器上运行一个nacos，还可以运行多个nacos，在服务器上搭建nacos集群环境，可以查看[官方说明](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)

![](https://picture.xcye.xyz/image-20220218233051767.png)



上图中下面部分的Nacos比便是三个nacos集群，SLB这里是内网，我们可以想成是Nginx



默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的Nacos节点，数据存储是存在一致性问题的。 

为了解决这个问题，Nacos采用了集中式存储的方式来支持集群化部署，目前只支持MySQL的存储。 



## 三种模式

nacos支持三种模式，详细[请看](https://nacos.io/zh-cn/docs/deployment.html)

- 单机模式 - 用于测试和单机试用。
- 集群模式 - 用于生产环境，确保高可用。
- 多集群模式 - 用于多数据中心场景。



## 演示集群模式

这里使用一个Nginx，一个MySQL，三个nacos搭建一个nacos集群



### 1.执行sql

因为这里我们的数据，存储在MySQL中的，所以我们需要在MySQL中，创建所需的库和表，但是sql语句官方已经提供了，默认在`nacos/conf/`目录中

![](https://picture.xcye.xyz/image-20220218233758803.png)



### 2.编辑application.properties

此文件也是在conf目录，在该文件中，加入下面内容

```properties
spring.datasource.platform=mysql
 
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```



### 3.修改cluster.conf

![](https://picture.xcye.xyz/image-20220218234040484.png)

在该文件中，增加以下内容，这里演示三台nacos

![](https://picture.xcye.xyz/image-20220218234149361.png)

这个需要搭建几个nacos就写几个，注意这里得host不能使用localhost，上面的配置，也就是部署三台nacos机器，分别运行在3333,4444,5555这三个端口上



### 4.编辑startup.sh

编辑Nacos的启动脚本startup.sh，使它能够接受不同的启动端口，我们编辑的目的就是为了在执行`startup.sh`的时候，我们能够传入额外的参数，从而在不同端口启动

![](https://picture.xcye.xyz/image-20220218234513113.png)

![](https://picture.xcye.xyz/image-20220218234535755.png)



### 5.配置Nginx

配置Nginx的话，自行解决，也就是配置负载均衡，然后就可以了，这个就是搭建nacos集群模式的步骤

