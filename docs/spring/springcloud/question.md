---
date: 2022/1/15 19:30
tag: [mybatis,spring,spring boot,spring cloud]
---

# 学习spring cloud过程中的问题记录

这里记录了我学习spring cloud的过程中，遇到的一些错误，以及一些已经忘记的知识点



## mybatis

### useGeneratedKeys

根据官方的解释

>  允许JDBC支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）

我们可以在三个地方设置该参数

- 在 `settings` 元素中设置 **useGeneratedKeys** 参数
- 在 `xml` 映射器中设置 **useGeneratedKeys** 参数
- 在 `接口` 映射器中设置 **useGeneratedKeys** 参数

详细可以看此篇[博客](https://blog.nowcoder.net/n/efa0db7ba0ba48fab13e7a6209a0c12d?from=nowcoder_improve)

这个参数的作用就是，比如我们执行`insert`操作，那么我们能够拿到数据库自动生成的主键id

`keyProperty`参数是和`useGeneratedKeys`一起使用的，除了这个，还有`keyColumn`，他们的区别是

- `keyColumn` 数据库 列名（or 别名）
- `keyProperty` java 中要封装的 类的参数名

![](https://picture.xcye.xyz/image-20220115194704947.png)



### resultMap标签

详细请看mybatis的[笔记](../../mybatis/readme#resultMap)



## spring

### service

当我们在service的实现类上，注入dao的对象的时候，可以使用spring的`Autowired`，我们也可以使用java自带的`Resource`进行依赖注入

```java
@Resource
private PaymentDao paymentDao;
```





### 使用restTemplate传递数据，收不到

```java
public CommonResult create(Payment payment) {}
```

最初是这样的，我们从另一个服务中，使用`restTemplate`的方法请求该controller的时候，会发现payment中的数据，没有

```java
restTemplate.postForObject(serviceUrl.getPaymentUrl() + "/payment/create",payment,CommonResult.class);
```

![](https://picture.xcye.xyz/image-20220115221158286.png)

出现这个原因，是需要我们使用`@RequestBody`这个注解

```java
public CommonResult create(@RequestBody Payment payment) {}
```

> `@RequestBody`主要用来接收前端传递给后端的`json字符串`中的数据的(请求体中的数据的)；
>
> 这里应该是`postForObject`方法传递过去的是一个json字符串，所以就导致收不到数据



### @Value注解

[SpringBoot之Spring@Value属性注入使用详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/99272510)

该注解，可以通过`key`获取到`application.yml`中的配置项值

使用

```yml
server:
	port: 8001
```

```java
@Value("${server.port}")
String serverPort;
```

使用的时候，一定要通过`${}`去获取



## maven

### 解决jdk版本一直改变

增加下面这个插件

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```





## cloud

### No instances available for CLOUD-PAYMENT-SERVICE

如果搭建集群的时候，请求报这个错误，那么大概率是`spring.application.name`为`CLOUD-PAYMENT-SERVICE`的服务提供者，没有被注册到注册中心

![](https://picture.xcye.xyz/image-20220116220155126.png)

> 一定要保证，此`CLOUD-PAYMENT-SERVICE`在上图箭头处



