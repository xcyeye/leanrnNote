---
date: 2022/1/16 19:54
---

# Eureka

 在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

## 什么是服务注册与发现
Eureka采用了CS的设计架构，`Eureka Server`作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过`Eureka Server`来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息，比如服务地址通讯地址等以别名方式注册到注册中心上。另一方（消费者|服务提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址))



![](https://picture.xcye.xyz/image-20220116195706119.png)

> 右图是`dubbo`



## Eureka包含两个组件：Eureka Server和Eureka Client

- `Eureka Server`提供服务注册服务
  各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

- `Eureka Client`通过注册中心进行访问
  它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）



## 服务端

`Eureka`的1版本和2版本的依赖区别有点大

```xml
<!--以前的老版本（当前使用2018）-->
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

```
<!--现在新版本（当前使用2020.2）-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

> 他们最主要的区别就是，在1版本中，客户端(`client`)和服务端(`server`)都是使用同一个jar，这样就使得客户端和服务端的依赖都是在同一个里面
>
> 但是在2版本中，服务端和客户端的依赖是分开的，看上图就知道



### 一、引入依赖

对于服务端，我们只需要在`pom.xml`文件种，引入服务端的依赖就行

```xml
<!--现在新版本（当前使用2020.2）-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### 二、修改配置文件

```yml
server:
  port: 7001

eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
    #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

我们需要在`application.yml`文件中，添加上面的配置



### 三、主启动

除此以外，我们还需要在spring的主方法类上，添加`@EnableEurekaServer`注解

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```



然后运行，我们打开`localhost:7001`便会看到下面的页面

![](https://picture.xcye.xyz/image-20220116201441013.png)





## 客户端

### 一、引入依赖

```xml
<!--eureka-client-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 二、修改配置文件

```yml
eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

> 这里得注意`register-with-eureka`项



### 三、添加注解

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentService8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentService8001.class,args);
    }
}
```

> 服务端和客户端的注解是不同的



当启动了之后，我们就可以在服务端看到我们刚添加的这个应用

![](https://picture.xcye.xyz/image-20220116201825610.png)





![](https://picture.xcye.xyz/image-20220116201908573.png)

> 为了方便，我们最好在每个应用的配置文件中，指定`spring.application.name`值





## 搭建集群

使用`Eureka`搭建集群，其原理图就是如下这种

![](https://picture.xcye.xyz/image-20220116220327982.png)

> 在`Eureka`服务注册中心中，有多个`Eureka Server`和多个`Service Provider`，当我们使用`Service Consumer`向服务提供者发送请求的时候，因为服务提供者全部都是注册在注册中心的，所以最终服务消费者是向注册中心中的某一个服务提供者发送请求，服务提供者我们可以设置采用轮训的方式，其中的逻辑，不需要我们自己写，这些都是`Eureka`自己做的事，我们要做的就是将服务提供者注册到注册中心，便可以了



微服务RPC远程服务调用最核心是：高可用，所以我们就需要搭建Eureka注册中心集群 ，实现负载均衡+故障容错



### Eureka Server



#### 修改映射关系

因为在`application.yml`中需要填写`eureka.instance.hostname`，这里Server我们搭建两个，7001和7002，为了方便，我们将localhost分别映射为`eureka7001.com`和`eureka7002.com`

```
127.0.0.1  eureka7001.com
127.0.0.1  eureka7002.com
```

> 当映射之后，有时候windows会抽风，会将我们的映射关系注释掉，所以需要注意



#### 修改配置

因为我们的`Eureka Server`存在多个，我们需要让他们能够相互监控着，也就是(两个举例)，7001中有7002,7002中有7001

![](https://picture.xcye.xyz/image-20220116222119976.png)

![](https://picture.xcye.xyz/image-20220116222153616.png)





所以最终的配置文件就应该是

```yml
server:
  port: 7001


eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7002.com:7002/eureka/
```

```yml

server:
  port: 7002


eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/
```

> 如果只有一个`Eureka Server`，那么`defaultZone`就是其本身，比如只有7001，那么`defaultZone`就是`defaultZone: http://eureka7001.com:7001/eureka/`



### Eureka Client

这里我们需要搭建两个`Server Provider`和一个`Consumer`，他们三个都需要注册到服务注册中心中（有两个）

搭建`Eureka Client`的时候，我们只需要修改`application.yml`便可以，配置如下，80(consumer)，8001(server provider)，8002(server provider)都是类似的

```yml

server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: 123456


eureka:
  client:
    #表示是否将自己注册进EurekaServer默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  # 集群版

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.atguigu.springcloud.entities    # 所有Entity别名类所在包
```



> 注意：
>
> 服务消费者，提供者搭建集群的思想，都是使他们的`spring.application.name`名字相同，所以这里，我们需要保证两个服务提供和的`spring.application.name`相同
>
> 因为我们搭建了两个`Eureka Server`，所以这里的`eureka.client.service-url.defaultZone`需要填三个，同理，有多个`Eureka Server`，那就填多个
>
> 服务提供者单机版和集群版的区别也就是在这里



#### 启动顺序

启动顺序也是一个讲究，在启动的时候，我们需要先启动`Eureka Server`，然后在启动服务提供者，其次才是服务消费者



最终的服务注册中心的页面如下

![](https://picture.xcye.xyz/image-20220116223526195.png)

![](https://picture.xcye.xyz/image-20220116223540186.png)



### 测试

那现在全部都搭建完成了，我们单独请求8001和8002的接口都是正常的，那么我们通过服务消费者去调服务提供者的时候，会发生什么呢

按理说，有两个服务提供者，8001和8002，那么我们通过消费者去调用，到底最终调用的是哪一个？

1. 访问`http://localhost/consumer/payment/get/1`

   能成功，但是无论怎么刷新，都是8001的接口

   出现这个问题，是因为我们在80中的controller中，restTemplate的url都是写死的，也就是8001的url，这里我们需要修改成`http://服务提供者名字`

   这里的服务提供者名字就是`spring.application.name`的名字，但是是大写，像这样`http://CLOUD-PAYMENT-SERVICE`

   ![](https://picture.xcye.xyz/image-20220116224030341.png)



2. 修改之后，再次发送请求，出现不知道的主机`http://CLOUD-PAYMENT-SERVICE`，这是因为，注册中心中有两个服务提供者，但是消费者去请求的时候，不知道该向这两个中的哪一个发送请求，我们需要设置采用轮训的方法（默认没有），需要在注册`RestTemplate`组件上，加上`LoadBalanced`注解

   ```java
   @Bean
   @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
   public RestTemplate getRestTemplate() {
       return new RestTemplate();
   ```

3. 测试通过

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能了。

> `Ribbon`也就是`RestTemplate`这个类的名称



![](https://picture.xcye.xyz/image-20220116224738760.png)





## 主机名称和访问信息有IP信息提示

![](https://picture.xcye.xyz/image-20220116225726564.png)

我这里，没有修改前是这样的，主机名称，也就是图中红框部分，我们修改成其他的字符串，比如`localhost:8002`这种，方便以后维护，访问信息有ip提示，就是鼠标移到主机名称上去，我们可以在左下角看到ip地址

![](https://picture.xcye.xyz/image-20220116230143261.png)

![](https://picture.xcye.xyz/image-20220116230206795.png)





## 服务发现

对于注册进`Eureka`中的微服务，我们可以通过服务发现来看到服务的信息，这个需要`DiscoveryClient`类的支持

```java
@GetMapping("/discovery")
public Object getDiscovery() {
    Map<String,Object> map = new HashMap<>();
    List<String> services = discoveryClient.getServices();
    for (String service : services) {
        log.info("------>{}",service);
        List<ServiceInstance> instances = discoveryClient.getInstances(service);
        map.put(service,instances);
    }
    return map;
}
```

![](https://picture.xcye.xyz/image-20220116231905634.png)

> 这个是很重要的事情



除此之外，我们还需要在对应模块的主启动类上，加入`EnableDiscoveryClient`注解

```java
@EnableDiscoveryClient //服务发现
@SpringBootApplication
@EnableEurekaClient
public class PaymentService8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentService8001.class,args);
    }
}
```





## Eureka自我保护


保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式：
EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. 
RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE 

 ### 为什么会产生Eureka自我保护机制？

为了防止EurekaClient可以正常运行，但是与EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除

### 什么是自我保护模式？

默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。


在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

> 这个也就是说，当我们的某个已经在注册中心注册的客户端，宕机了，或者因为网络原因不能使用了，该客户端就不能正常的向服务端发送心跳(`默认是90秒`)，那么服务端没有收到该客户端发送的心跳，其就会进入自我保护模式，也就是不会立马从注册中心中，移除该客户端，会有一个时间，如果再该时间内，客户端还是没有上线，那么服务端才会从注册中心那里移除



![](https://picture.xcye.xyz/image-20220117140040214.png)

 

 关闭或者开启自我保护模式，是通过`eureka.server.enable-self-preservation=true`，在服务端的配置文件中

如果我们关闭之后，那么我们在服务端，就会看到下图这样的效果

![](https://picture.xcye.xyz/image-20220117140304749.png)

 

 配置项还有

- `eureka.instance.lease-renewal-interval-in-seconds=30`
- `eureka.instance.lease-expiration-duration-in-seconds=90`

 

 

 

 

 
