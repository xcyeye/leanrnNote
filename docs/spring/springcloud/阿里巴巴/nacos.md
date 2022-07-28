---
date: 2022/2/6 10:37
title: 阿里巴巴nacos注册中心
---



- 为什么会叫这个名字？

  前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service。



nacos是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台，他是一个注册中心和配置中心的集合，可以这样理解

> nacos=spring cloud config + spring cloud bus + eureka

所以我们可以看到他的强大，并且其并不像eureka那样，需要我们自己创建一个模块，然后访问，才能看到web端，nacos我们只需要在服务器上或者本地运行，就可以通过8848端口进行访问



各种注册中心的比较

![](https://picture.xcye.xyz/image-20220218205721327.png)

> 据说 Nacos 在阿里巴巴内部有超过 10 万的实例运行，已经过了类似双十一等各种大型流量的考验



## 安装nacos

1. 到[GitHub](https://github.com/alibaba/nacos/releases)去下载
2. 解压，然后进入bin目录，运行`startup.sh -m standalone`
3. 在浏览器中访问8848端口

> 运行的时候，记得加上`-m standalone`参数，表示运行单一模式，如果直接运行`startup.sh`的话，会出错

登录的时候，默认的账号为nacos，密码为nacos



> 如果mac启动nacos报下面错误
>
> nohup: /Library/Internet: No such file or directory
>
> 这是因为mac初始化的时候，就安装了一个jdk，我们通过`echo $JAVA_HOME`，发现控制台没有任何的输出，所以我们需要在`~/.bash_profile`中，手动设置一下java环境
>
> ```sh
> export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home
> 
> 
> 
> export PATH=$JAVA_HOME/bin:$PATH
> ```
>
> 然后就可以了

## 服务提供者

我们演示服务提供者，还有服务消费者一起存在的情况，服务提供者被注册到nacos注册中心，服务消费者从注册中心处进行消费

我们在使用`spring cloud alibaba`的时候，需要导入一个依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
  <version>2.1.0.RELEASE</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

如果是一个微服务的话，那么我们直接在父`pom`中引入该依赖就可以了



### 依赖

在服务提供者中，还引入了下面依赖

```xml
<dependencies>
  <!--SpringCloud ailibaba nacos -->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  <!-- SpringBoot整合Web组件 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <!--日常通用jar包配置-->
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

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.75.128:8848 #配置Nacos地址 这里不能加上http

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

在配置的时候，我们一定要配置`spring.application.name`项，因为在注册中心中的名字就是使用该`name`值，所以一定要记得配置

我们将某个模块假如nacos注册中心，只需要配置`spring.cloud.nacos.discovery.server-addr`值便可以了



### 主启动类

主启动类上，除了加上spring注解之外，我们还需要加上`EnableDiscoveryClient `注解



### 业务类

为了演示方便，我们在建一个controller

```java
@RequestMapping("/payment")
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
```



然后再拷贝该9001模块，创建9002模块，他们只有端口不同，其余的都是相同的

然后启动这两个模块，启动成功之后，我们在nacos的服务管理中，便可以看到这两个实例

![](https://picture.xcye.xyz/image-20220218213000730.png)





## 服务消费者

### 依赖

```xml
<dependencies>
  <!--SpringCloud ailibaba nacos -->
  <dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  </dependency>
  <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
  <dependency>
    <groupId>xyz.xcye</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
  <!-- SpringBoot整合Web组件 -->
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <!--日常通用jar包配置-->
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

```yml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 172.16.180.129:8848


#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
```

> 这里的service-url.nacos-user-service是自己加入的配置项



### 业务类

因为这里得服务提供者是一个集群的模式，演示存在两个，所以我们服务消费者去调用服务提供者时，我们配置使用轮询的方式



```java
//controller
@RequestMapping("/consumer")
@RestController
public class OrderNacosController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;


    @GetMapping("/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}

//config
@Configuration
public class ApplicationContextBean {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

> 一定要记得在`RestTemplate`上加上`LoadBalanced`注解，否则调用接口访问的时候，会出现找不到host



### 测试

带服务消费者启动成功之后，我们访问`localhost:83/consumer//payment/nacos/{id}`，其最终便会去调用服务提供者，采用轮询的方式

然后注册中心到这里就结束了，复习的时候，记得对比一下多个注册中心的不同^_^





