---
date: 2022/1/20 21:02
tag: [spring cloud,spring,gateway]
---

# spring-gateway

[官网](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/)

## 这是什么？

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关；
但是在spring boot2中，spring公司便推出了spring-cloud-gateway，注意这个叫法，网关(gateway)和我们在spring-cloud-gateway中指的gateway是不一样的

![](https://picture.xcye.xyz/image-20220121192748552.png)

![](https://picture.xcye.xyz/image-20220121193710753.png)

> 网关是挡在所有设备发送请求之前的，如果有nginx的话，那么也可以在nginx之后，网关能够对请求进行过滤还有断言，路由分发等



`SpringCloud Gateway`作为Spring Cloud生态系统中的网关，目标是替代 Zuul，在Spring Cloud 2.0以上版本中，没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty。



> SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。
>
> 所以你能在springcloudgateway文档中，看到一段话
>
>  Spring Cloud Gateway requires the Netty runtime provided by Spring Boot and Spring Webflux. It does not work in a traditional Servlet Container or when built as a WAR.
>
> 也就是说，使用spring cloud gateway，我们必须引入`netty`，但是这个已经包含在了spring cloud gateway依赖中



![](https://picture.xcye.xyz/image-20220121193448971.png)

## 能干什么

spring cloud gateway能够反向代理，鉴权，流量控制，熔断，日志监控等功能

比如当请求的url中，没有携带我们需要的cookie信息的时候，我们能够直接返回错误内容



## 为什么需要gateway，而不用zuul？

1. neflix不太靠谱，zuul2.0一直跳票，迟迟不发布

    一方面因为Zuul1.0已经进入了维护阶段，而且Gateway是SpringCloud团队研发的，是亲儿子产品，值得信赖，而且很多功能Zuul都没有用起来也非常的简单便捷，Gateway是基于异步非阻塞模型上进行开发的，性能方面不需要担心。虽然Netflix早就发布了最新的 Zuul 2.x，但 Spring Cloud 貌似没有整合计划。而且Netflix相关组件都宣布进入维护期

   还有一个原因就是spring想要自己出一个，通过借鉴zuul的思想

2. SpringCloud Gateway的特性好

   基于Spring Framework 5, Project Reactor 和 Spring Boot 2.0 进行构建；
   动态路由：能够匹配任何请求属性；
   可以对路由指定 Predicate（断言）和 Filter（过滤器）；
   集成Hystrix的断路器功能；
   集成 Spring Cloud 服务发现功能；
   易于编写的 Predicate（断言）和 Filter（过滤器）；
   请求限流功能；
   支持路径重写。



### 他们的区别

在SpringCloud Finchley正式版之前，Spring Cloud推荐的网关是Netflix提供的Zuul：

1、Zuul 1.x，是一个基于阻塞 I/ O 的 API Gateway

2、Zuul 1.x 基于Servlet 2. 5使用阻塞架构它不支持任何长连接(如 WebSocket) Zuul 的设计模式和Nginx较像，每次 I/ O 操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx 用C++ 实现，Zuul 用 Java 实现，而 JVM 本身会有第一次加载较慢的情况，使得Zuul 的性能相对较差。

3、Zuul 2.x理念更先进，想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。 Zuul 2.x的性能较 Zuul 1.x 有较大提升。在性能方面，根据官方提供的基准测试， Spring Cloud Gateway 的 RPS（每秒请求数）是Zuul 的 1. 6 倍。

4、Spring Cloud Gateway 建立 在 Spring Framework 5、 Project Reactor 和 Spring Boot 2 之上， 使用非阻塞 API。

5、Spring Cloud Gateway 还 支持 WebSocket， 并且与Spring紧密集成拥有更好的开发体验

 

> 在当前的高并发情况下，项zuul1这种阻塞架构已经不能满足我们的需求，如果流量大，那么我们的系统，可能会随时搞崩



### zuul模型

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

servlet由servlet container进行生命周期管理。
container启动时构造servlet对象并调用servlet init()进行初始化；
container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service()。
container关闭时调用servlet destory()销毁servlet；

![](https://picture.xcye.xyz/image-20220121194503582.png)

### GateWay模型

在gateway中，使用到了[webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-new-framework)

传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的。
但是在Servlet3.1之后有了异步非阻塞的支持。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程（Spring5必须让你使用java8）

Spring WebFlux 是 Spring 5.0 引入的新的响应式框架，区别于 Spring MVC，它不需要依赖Servlet API，它是完全异步非阻塞的，并且基于 Reactor 来实现响应式流规范。





## 三大概念

### Route(路由)

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

### predicate(断言)

参考的是Java8的java.util.function.Predicate
开发人员可以匹配HTTP请求中的所有内容(例如请求头或请求参数)，如果请求与断言相匹配则进行路由



### Filter(过滤)

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。



> 一定要注意，在spring cloud gateway中，路由，断言和过滤，他们是`且`的关系，也就是说，有一个请求，只有此请求的url和cookie等满足我们设置的`spring.cloud.gateway.routes.predicates`中的条件的时候，此请求才会去请求真正的`spring.cloud.gateway.routes.uri`



![](https://picture.xcye.xyz/image-20220121195152568.png)

## 工作流程

![](https://picture.xcye.xyz/image-20220121195319923.png)

客户端向Spring Cloud Gateway发出请求(`这个请求的地址是公开的，但是最终的请求的地址，是隐藏在gateway中`)，然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler。

> 这里`与请求相匹配的路由`是指：
>
> `spring.cloud.gateway.routes.predicates`中设置的条件，如果满足此`predicates`设置的`所有条件`，那么gateway便会去请求真正的地址，这个真正的地址是设置在`spring.cloud.gateway.routes.url`中的



Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，
在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

> 其核心逻辑就是
>
> 路由转发+执行过滤器链



在执行的过程中，这个过滤器，推荐使用自定义过滤器，这样能够更满足我们的需要



> 如果客户端请求的url不存在，或者存在但是不满足设置的`spring.cloud.gateway.routes.predicates`，那么就会返回一个404 notfound页面，如果`spring.cloud.gateway.routes.predicates`中的条件满足的情况下，不满足过滤器，那么此时就不会返回404，这个需要看我们在自定义过滤器中，编写的逻辑，在自定义过滤器中，我们可以操作请求参数，cookIe的所有与`request`和`response`对象相关的信息，关键就看我们在过滤器中，返回的是什么



## 开始

因为`spring cloud gateway`是针对于微服务的，所以我们需要有注册中心，服务提供者等等

### 依赖

```xml
<dependencies>
    <!--gateway-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <!--eureka-client-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    <dependency>
        <groupId>com.atguigu.springcloud</groupId>
        <artifactId>cloud-api-commons</artifactId>
        <version>${project.version}</version>
    </dependency>
    <!--一般基础配置类-->
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



### 配置文件

```yml

server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**         # 断言，路径相匹配的进行路由
            #- After=2022-01-20T23:55:44.445+08:00[Asia/Shanghai]
            #- Cookie=chocolate, ch.p

        - id: payment_routh2 #payment_route    #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001          #匹配后提供服务的路由地址
          uri: lb://CLOUD-PAYMENT-SERVICE
          predicates:
            - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
            - After=2022-01-20T22:55:44.445+08:00[Asia/Shanghai]
eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

> `spring.cloud.gateway.uri`我们可以写具体的url，如(http://localhost:8001,https://aurora.xcye.xyz)等，也可以写我们服务的提供者名称，但是需要保证，我们在`application.yml`文件中，配置了`eureka.client`项，这里注册中心，不一定是`eureka`，指定固定的uri显然是不正确的方法，因为我们请求的时候，希望是负载均衡模式，所以uri可以写成
>
> ```
> lb://服务提供者名字
> ```
>
> 这里不能省略`lb://`或者不能改成`http://服务提供者名字`，lb就是`load balance`的意思，就好记了



因为我们配置了`eureka`，所以别忘了，在主启动类上使用`@EnableEurekaClient`激活哟



当配置好上述之后，我们启动配置中心(7001)和两个服务提供者（8001,8002），还有gateway（9527），在eurekaweb端中，可以看到他们都成功注册了，现在我们可以测试一下请求

比如此配置

```yml
uri: http://localhost:8001 #匹配后提供服务的路由地址
predicates:
- Path=/payment/get/**   
```

> 因为使用了gateway，所以服务提供者的url是隐藏的，我们需要通过gateway的端口去调用



- 服务提供者8001和8002的controller中，分别有存在`localhost:8001/payment/get/1`和`localhost:8002/payment/get/1`请求

所以我们需要通过暴露的gateway去调用服务提供者的api，也就是`localhost:9527/payment/get/1`，因为gateway的配置中，只有一个匹配Path的(`- Path=/payment/get/**`),所以很明显，这个localhost:9527/payment/get/1请求是满足`predicates`条件，所以gateway会将`/payment/get/1`部分追加到`uri`上，并发送请求，所以最终请求的链接就是`localhost:8001/payment/get/1`，如果你在浏览器中，查看过network，会发现他们并不是重定向操作



## 配置

我们可以在`application.yml`中进行配置，也可以通过代码方式进行配置，但是会很麻烦，还是配置文件香

```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        RouteLocatorBuilder.Builder routes = builder.routes();

        routes.route("path_baidu_guoNei",r -> r.path("/guonei").uri("https://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```



## 动态路由

如果我们将`spring.cloud.gateway.uri`写死，那么如果注册中心中，存在多个provider，请求并不会负载均衡，所以我们就需要使用动态路由，负载均衡方式可以是轮训，随机，或者自定义等等，像下面这样配置就行



```yml
uri: lb://服务提供者名字
```

> 注意是`lb`



## Predicate的使用

当我们启动9527，在日志打印那里，会发现

![](https://picture.xcye.xyz/image-20220121202933066.png)

详细查看官方文档

![](https://picture.xcye.xyz/image-20220121203033006.png)

有11种



## Filter的使用

![](https://picture.xcye.xyz/image-20220121203117821.png)

官方已经提供了31种，但是我们推荐，使用自定义过滤器

### 自定义过滤器

```java
@Slf4j
@Component
public class MyGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        String aurora = exchange.getRequest().getQueryParams().getFirst("aurora");
        log.info("执行自定义的全局过滤器: {},param: {}", new Date(),aurora);
        if (aurora == null) {
            log.warn("非法登录");
            exchange.getResponse().setStatusCode(HttpStatus.PARTIAL_CONTENT);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

创建一个类，实现`GlobalFilter, Ordered`两个接口，加入到容器中

`getOrder()`这个方法不用管，就是一个容器



## 官方文档解读

![image-20220324084242297](https://picture.xcye.xyz/image-20220324084242297.png)
