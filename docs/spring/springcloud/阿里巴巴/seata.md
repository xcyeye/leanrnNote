# seata

使用的话，我就不用讲了，直接看官方的教程

https://gitee.com/itCjb/spring-cloud-alibaba-seata-demo#https://gitee.com/link?target=https%3A%2F%2Fgithub.com%2Fseata%2Fseata%2Ftree%2Fdevelop%2Fscript%2Fclient

## 使用nacos作为配置中心

如果我们需要使用nacos作为配置中心的话，也就是我们将`registry.conf`中的type设置为`nacos`之后，我们需要配置，否则就会出现问题

![image-20220413145833453](https://picture.xcye.xyz/image-20220413145833453.png)

```java
io.seata.common.exception.FrameworkException: No available service
    
can not get cluster name in registry config 'service.vgroupMapping.aurora-message-seata-service-group', please make sure registry config correct
```



1. 设置`conf/registry.conf`中的type值，将其设置为nacos，并设置端口ip等

2. 因为使用nacos作为配置中心，就需要我们将配置都上传到nacos，官方提供了一个工具，我们可以直接一键上传

3. https://github.com/seata/seata/blob/develop/script/config-center/nacos/nacos-config.sh将此文件下载下来，放在seata的conf目录

4. https://github.com/seata/seata/blob/develop/script/config-center/config.txt将此文件下载下来，放在`seata`根目录下，也就是bin同级目录那里，不能修改该文件的名字，这个文件里面的内容，就是我们上传到nacos中的

    因为有一些配置是不需要的，我们可以删除，如果此文件中，某些配置项的值为空，并且没有注释掉，那么是不能上传成功的

5. 在`nacos-config.sh`目录下，运行`sh nacos-config.sh -h localhost -p 8848`

    ![image-20220413162657186](https://picture.xcye.xyz/image-20220413162657186.png)

    像下图这样就可以了，如果有一个错误，都会导致上传失败，对于造成上传失败的项，如果我们不需要的话，可以直接注释，然后再次上传



然后回到nacos刷新，就可以看到这些配置

![image-20220413162813402](https://picture.xcye.xyz/image-20220413162813402.png)

这些配置的命名都是以`seata的配置名作为名字`，值就直接写

![image-20220413163233065](https://picture.xcye.xyz/image-20220413163233065.png)



然后就可以了

### 使用

在`bootstrap.yml`文件中，设置nacos配置中心的地址

```yml
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
```

在`application.yml`文件中，在配置一下

```yml
seata:
  enabled: true
  enable-auto-data-source-proxy: true
  tx-service-group: default_tx_group
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      group: "SEATA_GROUP"
      namespace: ""
      username: "nacos"
      password: "nacos"
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      group: "SEATA_GROUP"
      namespace: ""
      username: "nacos"
      password: "nacos"
  service:
    vgroup-mapping:
      default_tx_group: default
  client:
    undo:
      log-table: undo_log
```

> 需要注意的是，如果我们的nacos配置中心和`application.yml`文件中的配置重合了，那么会使用`nacos`的配置作为默认配置，也就是`application.yml`会不生效



> 如果在使用`feign`进行服务之间的远程调用的时候，如果对异常进行了处理，一定要设置响应码为5xxx，否则如果远程服务发生了异常，那么也是不能回滚的



## mysql中使用dateTime问题

将mysql中的时间存储格式修改为timesmap，添加依赖

```
<dependency>
<groupId>com.esotericsoftware.kryo</groupId>
<artifactId>kryo</artifactId>
<version>2.24.0</version>
</dependency>

<dependency>
<groupId>com.esotericsoftware</groupId>
<artifactId>kryo</artifactId>
<version>4.0.2</version>
</dependency>

<dependency>
<groupId>de.javakaffee</groupId>
<artifactId>kryo-serializers</artifactId>
<version>0.44</version>
</dependency>
```

在`nacos`中增加一个配置

```
client.undo.logSerialization=kryo
```

![image-20220423125359311](https://picture.xcye.xyz/image-20220423125359311.png)





## 报错信息

can not get cluster name in registry config 'service.vgroupMapping.account-service-fescar-service-group', please make sure registry config correc

![image-20220503162849313](https://picture.xcye.xyz/image-20220503162849313.png)

因为之前的tx-service-group的值为`aurora_blog_tx_group`，需要修改成`aurora-blog-tx-group`，nacos中也需要修改一下

![image-20220503162954948](https://picture.xcye.xyz/image-20220503162954948.png)

issue

https://github.com/seata/seata-samples/issues/408
