---
date: 2022/1/18 12:07
tag: [spring boot,spring cloud,ribbon,微服务]
---

# Ribbon负载均衡服务调用

## 是什么

`Spring Cloud Ribbon`是基于Netflix Ribbon实现的一套客户端`负载均衡的工具`。

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的`软件负载均衡算法`和`服务调用`。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出`Load Balancer`（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

 

 [官网](https://github.com/Netflix/ribbon/wiki/Getting-Started)

 但是目前其已进入维护阶段，以后可以使用`Spring Cloud LoadBalancer`进行替换



 ## 什么是LB负载均衡(Load Balance)

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）。
常见的负载均衡有软件Nginx，LVS，硬件 F5等。

### Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别

- Nginx是服务器负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。

  也就是说，nginx是外层，但是ribbon是内层的负载均衡

-  Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用技术。

 

 ![](https://picture.xcye.xyz/image-20220118131845402.png)



LB负载均衡又分集中式LB和进行内LB

 ### 集中式LB

即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；

 ### 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

 

### 案例

比如我们运行着两个相同的服务，8001和8002，这两个服务都提供相同的接口调用，服务名是相同的(加入为spring-cloud-provider)

有一个服务消费者80端口，我们通过80端口(`http://spring-cloud-provider`)去调用8001和8002这两个服务提供的接口，并且设置轮训方式

那么我们不断的刷新浏览器，80端口这个消费者返回的信息，就是在8001和8002之间进行访问





## 架构

![](https://picture.xcye.xyz/image-20220118132431857.png)

> `Ribbon`是在服务消费者上的，不是在服务提供者上，一定要记得，他是客户端组件



## 依赖

![](https://picture.xcye.xyz/image-20220118132635325.png)

在引入eureka依赖的时候，就已经引入了

## RestTemplate

这个对象里面，我们常用的方法有两种

- 返回对象为响应体中数据转化成的对象，基本上可以理解为Json

  ![](https://picture.xcye.xyz/image-20220118132820429.png)

- 返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等

  ![](https://picture.xcye.xyz/image-20220118132827565.png)



 ## Ribbon核心组件IRule

根据特定算法中从服务列表中选取一个要访问的服务

比如我们使用的轮巡方式，也就是此接口的一个实现类提供的

![](https://picture.xcye.xyz/image-20220118133135576.png)

 

- `com.netflix.loadbalancer.RoundRobinRule` 轮巡
- `com.netflix.loadbalancer.RandomRule` 随机
- `com.netflix.loadbalancer.RetryRule` 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务
- `WeightedResponseTimeRule` 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择
- `BestAvailableRule` 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
- `AvailabilityFilteringRule `先过滤掉故障实例，再选择并发较小的实例
- `ZoneAvoidanceRule` 默认规则,复合判断server所在区域的性能和server的可用性选择服务器



### 如何替换

1. 新建配置类

   ![](https://picture.xcye.xyz/image-20220118133433442.png)

   > 新建这个配置类的时候，一定要注意，不能够让该类被`@ComponentScan`注解扫描到，此注解在`@SpringBootApplication`注解中使用到，我们知道，在`spring`主启动类所在包及其子包中的类，都将被`@ComponentScan`扫描到，所以我们就要保证，我们新建的这个配置类，不能在主启动类所在包下及其子包下

   ![](https://picture.xcye.xyz/image-20220118133843337.png)

2. 在该`MySelfRule.java`中，添加以下内容

   ```java
   @Configuration
   public class MySelfRule {
       @Bean
       public IRule myRule() {
           //return new RandomRule();
           return new RetryRule();
       }
   }
   ```

3. 在主启动类上，使用`RibbonClient`注解，指定负载均衡规则

   ```java
   @EnableEurekaClient
   @SpringBootApplication
   @RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
   public class ConsumerMain80 {
       public static void main(String[] args) {
           SpringApplication.run(ConsumerMain80.class,args);
       }
   }
   ```

   > 这里的`name`填服务提供者的`spring.application.name`值，大写

4. OK



## 自定义负载均衡算法

我们可以参照已有的那些负载均衡算法，自己动手写一个，需要实现`IRule`接口

