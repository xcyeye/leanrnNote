---
date: 2022/1/17 15:21
tag: [spring boot,spring cloud,zookeeper]
---

# zookeeper服务注册与发现

因为`Eureka`在2版本应为某些原因停止更新了，所以我们不能够继续使用，[停更说明](https://github.com/Netflix/eureka/wiki)

我们可以使用`zookeeper`作为服务注册中心，使用zookeeper也是一样的方法，我们只需要将服务提供者注册到服务注册中心中，然后服务消费者直接去调这个服务就行了

- zookeeper是一个分布式协调工具，可以实现注册中心功能
- 需要在Linux中安装zookeeper
- 并且查看注册中心中，有哪些服务，都是通过Linux命令进行查看



## 安装zookeeper

1. 下载Linux包
2. 解压
3. 将`conf/zoo_sample.cfg`文件更名为`zoo.cfg`

4. 新建一个文件夹data，修改`conf/zoo.cfg`中的`dataDir=./data`项

5. 启动zookeeper服务端，`./bin/zkServer.sh`

6. 写代码，将服务提供者注册到注册中心中

7. 服务消费者通过路径`http://服务提供者注册到zookeeper注册中心的服务名`调用服务提供者

8. 启动zookeeper客户端，可以查看注册中心中，都有哪些注册的服务

   `bin/zkCli.sh`启动zookeeper客户端

   ```sh
   WatchedEvent state:SyncConnected type:None path:null
   [zk: localhost:2181(CONNECTED) 0] ls
   ls [-s] [-w] [-R] path
   [zk: localhost:2181(CONNECTED) 1] ls /
   [zookeeper]
   [zk: localhost:2181(CONNECTED) 2] ls /zookeeper 
   [config, quota]
   [zk: localhost:2181(CONNECTED) 3] 
   ```

   `ls /`是查看所有的内容

   `ls /services`查看所有服务

   `get /services/服务名/流水号`得到某个服务的json字符串信息

9. 完成



## 依赖以及配置文件

### 服务提供者

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

```yml
#8004表示注册到zookeeper服务器的支付服务提供者端口号
server:
  port: 8004
#服务别名----注册zookeeper到注册中心名称
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 192.168.111.144:2181
```

> 如果我们需要搭建集群，那么`connect-string`使用逗号分割开，和`eureka`类似

- 主启动类

  ```java
  @SpringBootApplication
  @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
  ```

  主启动类上，需要加上`EnableDiscoveryClient`注解



### 服务消费者

- 依赖

  ```html
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
      <!--先排除自带的zookeeper-->
      <exclusions>
          <exclusion>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  <!--添加zookeeper3.4.9版本-->
  <dependency>
      <groupId>org.apache.zookeeper</groupId>
      <artifactId>zookeeper</artifactId>
      <version>3.4.9</version>
  </dependency>
  ```

  > 因为在`spring-cloud-starter-zookeeper-discovery`中，就有`zookeeper`依赖，如果再次引入的话，会存在依赖冲突，所以这里需要使用`exclusions`排除

- 配置

  ```yml
  server:
    port: 80
  
  spring:
    application:
      name: cloud-consumer-order
    cloud:
    #注册到zookeeper地址
      zookeeper:
        connect-string: 192.168.111.144:2181
  ```

  > 这里我们也是将这个消费者注册到zookeeper注册中心中，注册到哪个注册中心，只需要填写`connect-string`项就行

- 启动类

  消费者主启动类上，我们可以不用添加`EnableDiscoveryClient`注解

- 配置`RestTemplate`Bean

  ```java
  @Configuration
  public class ApplicationContextBean{
      @Bean
      @LoadBalanced
      public RestTemplate getRestTemplate()
      {
          return new RestTemplate();
      }
  }
  ```

  > 如果这里没有`LoadBalanced`注解的话，那么就会存在Unknown Host错误，这个注解也就是消费者调用服务提供者时使用负载均衡

- controller

  调用`restTemplate.getForObject()`方法，访问`http://zookeeper注册中心中的服务提供者名字`便可以

- 完成



上面这个就是使用zookeeper作为注册中心的过程，其实和eureka的使用是一样的，只是他们的底层逻辑不一样

都是通过将服务提供者注册到注册中心中，服务消费者根据自己的情况，然后服务消费者通过`http://服务提供者名字`便可以调用服务提供者的接口，如果是搭建集群，那么就将多个服务提供者注册到注册中心中，便可以



只是zookeeper和eureka有一点不同，zookeeper的服务端是需要在Linux中启动，而eureka需要我们自己写代码，在配置中，通过下面方式进行设置

```yml
#eureka
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://eureka7001.com:7001/eureka/
```



