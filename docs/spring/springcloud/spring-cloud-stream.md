---
date: 2022/2/18 12:28
title: spring cloud stream消息驱动
tag: [spring,spring-cloud]
---



我们在使用`spring-cloud-bus`的时候，会遇到一个问题，因为`bus`目前只支持`Kafka`和`rabbitmq`，但是如果我们的项目和其他的项目对接，这个和我们对接的项目使用了其他的消息中间件，那么这个时候，我们是不是还得去学一下这个消息中间件？

为了解决这个问题，我们可以使用`spring cloud stream`，这个是构建消息驱动微服务的框架，和我们使用的jdbc的原理差不多，我们使用的时候，不需要注重他底层的实现，只需要配置对应的消息中间件就行



应用程序通过`inputs`或` outputs`来与Spring Cloud Stream中binder对象交互。 通过我们配置来binding(绑定) ，而Spring Cloud Stream的binder对象负责与消息中间件交互。 

所以，我们只需要搞清楚如何与spring Cloud Stream交互就可以方便使用消息驱动的方式。 

Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，引用了发布-订阅、消费组、分区的三个核心概念。 

> 目前仅支持RabbitMQ、Kafka。 

中文文档可以参照[这个](https://m.wang1314.com/doc/webapp/topic/20971999.html)



## 设置思想



::: tip Why

如果我们使用的是`RabbitMQ`和`Kafka`，由于这两个消息中间件的架构上的不同， 像RabbitMQ有exchange，kafka有Topic和Partitions

这些中间件的差异性导致我们实际项目开发给我们造成了一定的困扰，我们如果用了两个消息队列的其中一种，后面的业务需求，我想往另外一种消息队列进行迁移，这时候无疑就是一个灾难性的， 一大堆东西都要重新推倒重新做 ，因为它跟我们的系统耦合了，这时候springcloud Stream给我们提供了一种解耦合的方式。 

:::



### 绑定器

在没有绑定器这个概念的情况下，我们的SpringBoot应用要直接与消息中间件进行信息交互的时候，由于各消息中间件构建的初衷不同，它们的实现细节上会有较大的差异性 ，`通过定义绑定器Binder作为中间层，完美地实现了应用程序与消息中间件细节之间的隔离 `。

Stream对消息中间件的进一步封装，可以做到代码层面对中间件的无感知，甚至于动态的切换中间件(rabbitmq切换为kafka)，使得微服务开发的高度解耦，服务可以关注更多自己的业务流程 



![](https://picture.xcye.xyz/image-20220218125510762.png)

> 这个绑定器是用来绑定消息容器的生产者和消费者，它有两种类型，INPUT和OUTPUT，INPUT对应于消费者，OUTPUT对应于生产者。



## 标准流程

![](https://picture.xcye.xyz/image-20220218125829530.png)



- Binder

  链接中间件，屏蔽差异

- Channel

  通道，是队列Queue的一种抽象，在消息通讯系统中就是实现存储和转发的媒介，通过Channel对队列进行配置

- Source和Sink

  可理解为参照对象是Spring Cloud Stream自身，从Stream发布消息就是输出，接受消息就是输入。



从该图的流程中，我们可以看出，stream处理不同消息中间件的流程是，生产者将消息输出到stream，stream将此生产者和我们的消费者进行绑定，这样消费者就能够得到该消息



## 常用注解

![](https://picture.xcye.xyz/image-20220218130335469.png)

![](https://picture.xcye.xyz/image-20220218130359317.png)



## 测试

这里我们搭建一个消息生产模块`cloud-stream-rabbitmq-provider8801`和两个消息接收模块，也就是消费者`cloud-stream-rabbitmq-consumer8802`和`cloud-stream-rabbitmq-consumer8803`

这里使用`eureka`作为注册中心

## 消息生产模块

### 依赖

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
  </dependency>
  <!--基础配置-->
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

```yaml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 192.168.75.128
                port: 5672
                username: admin
                password: 123456
      bindings: # 服务的整合处理
        output: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为json，文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: send-8801.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```

> 在配置`rabbitmq`的时候，host一定不能加上http





### 定义一个发送接口，并且实现该接口

```java
//发送接口
public interface MessageProvider {
    String send();
}


//实现类
@EnableBinding(value = Source.class)
public class MessageProviderImpl implements MessageProvider {

    @Resource
    private MessageChannel output;

    @Override
    public String send() {
        String message = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(message).build());
        return message;
    }
}
```

> 这里得注意，`output`是对应于生产者，`input`是对应于消费者



然后我们调用`send()`方法便可以将消息发送出去了



### controller

```java
public class SendMessageController {

    @Resource
    private MessageProvider provider;

    @GetMapping("/sendMessage")
    public String sendMsg() {
        String send = provider.send();
        log.info("发送 {}",send);
        return "成功发送";
    }
}
```



### 启动测试

当我们启动该消息生产模块的时候，那么我么便可以在web端的rabbitmq中看到一个新的交换机，交换机的名字也就是我们配置的`spring.cloud.stream.bindings.output.destination`





## 消费者模块

### 依赖

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  <!--基础配置-->
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

```yaml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: # 在此处配置要绑定的rabbitmq的服务信息；
        defaultRabbit: # 表示定义的名称，用于于binding整合
          type: rabbit # 消息组件类型
          environment: # 设置rabbitmq的相关的环境配置
            spring:
              rabbitmq:
                host: 192.168.75.128
                port: 5672
                username: admin
                password: 123456
      bindings: # 服务的整合处理
        input: # 这个名字是一个通道的名称
          destination: studyExchange # 表示要使用的Exchange名称定义
          content-type: application/json # 设置消息类型，本次为对象json，如果是文本则设置“text/plain”
          binder: defaultRabbit # 设置要绑定的消息服务的具体设置
          #group: aurora

eureka:
  client: # 客户端进行Eureka注册的配置
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳的时间间隔（默认是30秒）
    lease-expiration-duration-in-seconds: 5 # 如果现在超过了5秒的间隔（默认是90秒）
    instance-id: receive-8802.com  # 在信息列表时显示主机名称
    prefer-ip-address: true     # 访问的路径变为IP地址
```



### 业务类

```java
@Slf4j
@Component
@EnableBinding(value = Sink.class)
public class ReceiveMessageListener {

    @Value("${server.port}")
    private String port;

    @StreamListener(value = Sink.INPUT)
    public void inputMsg(Message<String> message) {
        String payload = message.getPayload();
        log.info("消费者1号 ---> 端口: {},message: {}",port,payload);
    }
}
```

我们只需要写上这个类，就可以了

当我们的消息生产者发送消息之后，那么使用`StreamListener`注解，并且value为`Sink.INPUT`的方法便会接收到该消息





## 消息重复消费和消息持久化



### 重复消费

当我们拷贝消费者模块，那么现在存在两个消费者模块，分别是8082和8083，但是当 生产者发送一个消息之后，我们会发现8082和8083都消费了该消息，这就会造成消息的重复消费，我们应该保证只有一个消费消费



造成这个的原因是因为，8082和8083他们都是属于不同的`group`，不是属于同一个分组的模块，他们都共同绑定在一个交换机上，那么这些不属于同一个分组的模块，都能够消费该消息，所以解决的办法就是，为8082和8083设置相同的分组，这样生产者生产消息之后，同一个分组中，只会有一个能够消费消息



- 如果没有设置`group`值，那么8082和8083队列的名称，会是下面这种形式

  ![](https://picture.xcye.xyz/image-20220218200947654.png)

- 设置`group`之后

  > 8082的`group`为aurora-8082
  >
  > 8083的`group`为aurora-8083

  ![](https://picture.xcye.xyz/image-20220218201205644.png)

如果我们设置了group字段，那么队列的名称也会变成`交换机名称.group名称`

该`group`是通过`spring.cloud.stream.bindings.input.group`进行设置的



### 持久化

现在我们去掉8082的group字段值，直接注释，8083不做任何处理，然后我们关闭8082和8083，调用消息提供者发送4条消息，成功发送之后，我们启动8082和8083，观察在启动的过程中，8082和8083的控制天会不会打出发送的这四条消息(`因为他们不属于同一个组，所以8082和8083都能够消费`)

但是最终就只有8083在控制台打印出生产的消息，8082没有

如果没有设置`group`，那么消息是不能持久化，也就是说，如果生产者生产了多条消息，设置group字段的某个模块突然挂机了，但是当该模块重新启动的时候，生产的消息能够被消费



