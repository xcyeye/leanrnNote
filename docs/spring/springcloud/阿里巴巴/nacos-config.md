---
date: 2022/2/18 21:51
title: nacos之配置中心
---

我们知道nacos是一个注册中心和配置中心的集合，并且配置中心并不像`spring cloud config`那样，配置文件需要借助GitHub，gitee等进行存放，访问速度低

nacos的配置文件，我们可以直接存放在数据库中，或者是其他的方式，反正不需要借助GitHub，所以，就很友好(^▽^)



## 使用nacos配置中心文件演示

这里我们创建一个模块，这个模块去获取nacos配置中心的配置文件



### 依赖

```xml
<dependencies>
  <!--nacos-config-->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
  </dependency>
  <!--nacos-discovery-->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  <!--web + actuator-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <!--一般基础配置-->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
```



### 配置项

这里需要创建两个yaml文件，`application.yml`和`bootstrap.yml`，关于他们两个的区别，去看`spring cloud config`处的笔记，那里有记录

```yml
# bootstrap.yml
# nacos配置
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.180.129:8848 #Nacos服务注册中心地址
      config:
        server-addr: 172.16.180.129:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置


  # ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
  
# application.yml
spring:
  profiles:
    active: dev # 设置开发环境
```



> 如果我们的配置文件是yaml后缀，那么`file-extension`我们不能简写成`yml`，一定要是`yaml`，否则会找不到这个配置文件



### 业务类

```java
@RefreshScope
@RestController
@RequestMapping("/config")
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

> 在控制器类加入`@RefreshScope`注解使当前类下的配置支持 *Nacos* 的动态刷新功能。





### nacos中加入配置文件

我们直接在nacos的web端，点击配置管理，新增一个就行

![](https://picture.xcye.xyz/image-20220218223754169.png)

这里我们需要注意`Data Id`的书写规范，否则在启动的时候，就会报错

请看[官方介绍](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)



这里我们增加一个yaml类型的配置

![](https://picture.xcye.xyz/image-20220218223937537.png)

> `Data Id`和`Group`发布了之后，就不能进行修改了

我们上面增加了一个`config.info`的配置项，那么我们现在可以直接在业务类中使用该配置项

```java
@Value("${config.info}")
private String configInfo;
```

> 一定要注意，这里得`config.info`并不是在`bootstrap.yml`和`application.yml`中配置的，这个是直接使用nacos配置中心的，一定要注意，在配置中心的配置项，我们可以直接使用，不然配置中心有何用



如果我们现在修改nacos中的`config.info`的值，那么我们不需要做额外的操作，直接访问，配置项便已经更新到最新的，并且nacos的配置中心支持回滚操作







## 配置中心分类管理

在实际开发的过程中，我们会遇到以下情况

1. 开发的时候，通常会准备

   - dev开发环境
   - test测试环境
   - prod生产环境

   如何保证指定环境启动时服务能正确读取到nacos上相应环境的配置文件？

2. 一个大型分布式服务系统会有很多微服务子项目，每个微服务项目又会有相应的开发环境，测试环境，预发环境，正式环境等，那怎么对这些为服务的配置进行管理？



那么解决这个问题，就可以使用nacos的`Data Id`和`Group`和`Namespace`三种方案进行管理

他们之间的区别和联系如下

他们三者相当于Java中的包名和类名，`最外层的namespace是可以用于区分部署环境的，Group和DataID逻辑上区分两个目标对象。`

![](https://picture.xcye.xyz/image-20220218225837673.png)

> 他们的默认值为：
>
> Namespace=public
>
> Group=DEFAULT_GROUP
>
> Cluster是DEFAULT



Nacos默认的命名空间是public，Namespace主要用来实现隔离。 比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。 

 Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去 

 Service就是微服务；一个Service可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。 

比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房， 这时就可以给杭州机房的Service微服务起一个集群名称（HZ）， 给广州机房的Service微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。 

最后是Instance，就是微服务的实例。 



### Data ID方案

我们可以使用默认的namespace和group新建不同的开发环境的配置文件，在nacos中，根据`${prefix}-${spring.profiles.active}.${file-extension}`新建不用环境的配置文件，然后我们只需要在模块配置文件中，指定`spring.profiles.active`项的值，便可以在test/prod/dev三个环境中进行切换(前提是nacos中新建了这三个开发环境的配置文件)



### Group方案

除了dataId，我们还可以在配置中心创建配置文件的时候，指定`group`的值，然后在`bootstrap.yml`文件中，将`spring.cloud.nacos.config.group`的值和web端设置的group相同



### Namespace方案

我们还可以创建Namespace，然后在`bootstrap.yml`中，将`spring.cloud.nacos.config.namespace`的值和web端设置的namespace相同

![](https://picture.xcye.xyz/image-20220218231001974.png)
