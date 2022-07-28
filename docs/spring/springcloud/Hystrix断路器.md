---
date: 2022/1/9 21:36
tag: [spring cloud,hystyix,微服务]
---

# Hystrix断路器

## 为什么需要Hystrix？

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败

![](https://picture.xcye.xyz/image-20220119213823616.png)

因为我们知道，如果一个消费者默认情况下，去调服务提供者的服务，如果超过3秒还是多少，那么就会报一个`read timeout`的错误，返回的可能就是错误html页面，这是我们不愿意看到的，还有可能服务消费者，服务提供者发生异常，那么也会导致服务调用失败，这样不利于使用该接口的程序使用，所以我们就需要Hystrix来解决这个问题



## what

`Hystrix`是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

`断路器`本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



### Do？

这个能够做什么？

1. 服务降级
2. 服务熔断
3. 接近实时的监控
4. .......



这里的服务降级，并不是我们普片认为的那样，降低等级的意思，服务降级，就是加入一个什么发生异常，超时等情况时，能够代替该程序执行的方法，一定要搞清楚

官网地址[为](https://github.com/Netflix/Hystrix/wiki/How-To-Use)，但是该框架目前已停更维护阶段



## 重要概念

### 服务降级

服务器忙，请稍后再试，不让客户端等待并立刻返回一个友好提示，fallback

哪些会触发服务降级

1. 程序运行异常
2. 超时
3. 服务熔断触发服务降级
4. 线程池/信号量打满也会导致服务降级



### 服务熔断

类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示

这个服务熔断，就比如保险丝一样

服务的降级->进而熔断->恢复调用链路



### 服务限流

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行



## 使用

我们可以在服务提供者上使用，可以在任何地方，在`service`的实现类上，在`controller`上，等等，并不需要在某个特定的地方使用

1. 新建module

2. 导入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

3. 修改配置文件

   ```yml
   server:
     port: 8001
   
   spring:
     application:
       name: cloud-provider-hystrix-payment
   
   eureka:
     client:
       register-with-eureka: true
       fetch-registry: true
       service-url:
         #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
         defaultZone: http://eureka7001.com:7001/eureka
   ```

   > 这里其实就是将该module注册到注册中心

4. 在主启动类上，加入`@EnableEurekaClient `，然后写个controller测试

可以使用`jmeter`工具，进行高并发测试

当没有使用jmeter工具进行高并发测试时，一切都正常，尽管有些controller我们设置了睡眠时间，但是服务提供者还是正常运行(服务消费者调用有睡眠时间的controller可能会存在读取超时异常)



但是使用jmeter工具之后，会发现原本没有睡眠时间的controller的请求，都变得非常慢，这还只是对服务提供者，如果存在服务消费者调用这些服务提供者的服务，那么很快，消费者便会出现读取超时错误，这是我们不愿意看到的，所以我们就需要使用到`降级/容错/限流`等技术



- 比如provider出现高并发，请求一个url的时间，变成差不多6秒左右，那么我们可以使用服务降级，当请求时间超过3秒时，我们就直接返回一个请求拥挤，稍后再试等信息，对于consumer，我们也可以同样设置
- 因为出现异常，也会出现服务降级，所以都是适用的

### 服务降级配置

1. 在需要的地方，使用`@HystrixCommand`注解

   > 注意是需要的地方，可以是service，controller等等

   比如我们这里，在一个service上使用

   ```java
   @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
       @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "2000")
   })
   public String paymentInfo_TimeOut(Integer id) {
       try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
       return "线程池:"+Thread.currentThread().getName()+"paymentInfo_TimeOut,id: "+id+"\t"+"O(∩_∩)O，耗费3秒";
   }
   
   public String paymentInfo_TimeOutHandler(Integer id) {
       return "paymentInfo_TimeOut方法超时了";
       //System.out.println("1111111111111");
   }
   ```

   > `fallbackMethod`是执行服务降级的方法，一定要保证，该方法和使用`@HystrixCommand`注解标注的方法的所有一样，包括修饰符，返回值，参数，除了方法名不一样外，逻辑也不用一样，一定要保证修饰符，返回值，参数一样
   >
   > 如果只是参数不一样，那么会出现`fallbackMethod`的方法找不到，如果返回值不一样，你可以测试一下O(∩_∩)O哈哈~
   >
   > `commandProperties`就是设置一些参数，就是什么条件触发服务降级，上面就是设置程序运行时间操作2秒，尽管设置了操作2秒，就发生服务降级，但是如果`paymentInfo_TimeOut(Integer id)`方法体中，出现异常，那么也会导致服务降级，尽管时间没有操作2秒

   

2. 在主启动类上，加入`@EnableCircuitBreaker`注解，一定要加入该注解，否则服务降级不会出现

现在你再使用jmeter工具测试高并发情况，就会发现，当请求时间操作2秒时，会直接返回`paymentInfo_TimeOutHandler`方法中的返回体

我们同样也可以在consumer服务使用`hustrix`

比如我们在consumer的controller上使用服务降级

```java
@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
    @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1000")
})
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
    /*String s = paymentHystrixService.paymentInfo_TimeOut(id);
        return s;*/
    int a = 10 / 0;
    return "sdf";
}

String paymentInfo_TimeOutHandler(@PathVariable("id") Integer id) {
    return "80方法调用超时了";
}
```

> 需要在consumer的配置文件上，加上
>
> ```yml
> feign:
>   hystrix:
>     enabled: true
> ```



::: warning

如果使用上述方式，那么会出现怎样的问题？

1. 每个业务方法对应一个兜底的方法，代码膨胀
2. 统一和自定义的分开

:::

   



## 问题解决

### 方法一

我们就需要解决上面每个方法都需要对应一个服务降级方法的问题

我们可以在对应方法的类上使用`@DefaultProperties`注解，设置默认的服务降级方法，这样可以为那些没有使用`@HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {...`指定特定的服务降级方法进行处理



```java
@DefaultProperties(defaultFallback = "defaultHandlerMethod")
@RestController
public class OrderHystirxController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    String paymentInfo_OK(@PathVariable("id") Integer id) {
        String s = paymentHystrixService.paymentInfo_OK(id);
        return s;
    }
    @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1000")
    })
    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
        String s = paymentHystrixService.paymentInfo_TimeOut(id);
        return s;
        //int a = 10 / 0;
        //return "sdf";
    }

    String paymentInfo_TimeOutHandler(@PathVariable("id") Integer id) {
        return "80方法调用超时了";
    }

    @GetMapping("/consumer/payment/hystrix/default_test")
    public String default_test() {
        int a = 10 /0;
        return "this is 80's default test method";
    }

    @HystrixCommand
    @GetMapping("/consumer/payment/hystrix/default_test_test")
    public String default_test_test() {
        int a = 10 /0;
        return "this is 80's default test method";
    }

    public String defaultHandlerMethod() {
        return "-----80------defaultHandlerMethod";
    }
}
```

`@DefaultProperties(defaultFallback = "defaultHandlerMethod")`中的`defaultFallback `就是指定默认的服务降级方法，但是需要注意的是，此`defaultHandlerMethod()`方法只是针对于那些使用了`@HystrixCommand`注解，但是在该注解中，没有设置`fallbackMethod`参数的那些，对于上面的`paymentInfo_TimeOut()`方法，出现服务降级，找的是`paymentInfo_TimeOutHandler`方法，而不是默认方法，一定要注意，默认的服务降级方法，只对存在`@HystrixCommand`注解的有效，上面代码中的`default_test`方法就没有效



而且还必须保证默认服务降级方法和发生服务降级的来源方法的返回值是一样的，如果发生服务降级的来源方法的返回值是`void`类型，但是默认的服务降级处理方法的返回值是`String`，那么也会抛出异常，所以，这个默认的处理方法，感觉并不是很好用，你得保证所有的服务降级来源方法的返回值和默认服务降级方法的返回值一样，但是如果我们在controller上进行设置，统一了controller的返回值类型，使用这个处理也是yyds

上面默认服务降级方法的返回值和发生服务降级来源方法的返回值不同，会报下面错误

::: details 详细代码

```java
@DefaultProperties(defaultFallback = "defaultHandlerMethod")
@RestController
public class OrderHystirxController {

    @Resource
    private PaymentHystrixService paymentHystrixService;

	//.............

    @HystrixCommand
    @GetMapping("/consumer/payment/hystrix/default_test_test")
    public void default_test_test() {
        int a = 10 /0;
        //return "this is 80's default test method";
        System.out.println("this is 80's default test method");
    }

    public String defaultHandlerMethod() {
        return "-----80------defaultHandlerMethod";
    }
}
```

:::



```java
There was an unexpected error (type=Internal Server Error, status=500).
Incompatible return types. Command method: public void xyz.xcye.constroller.OrderHystirxController.default_test_test(); Fallback method: public java.lang.String xyz.xcye.constroller.OrderHystirxController.defaultHandlerMethod(); Hint: Fallback method 'public java.lang.String xyz.xcye.constroller.OrderHystirxController.defaultHandlerMethod()' must return: void or its subclass
com.netflix.hystrix.contrib.javanica.exception.FallbackDefinitionException: Incompatible return types. 
Command method: public void xyz.xcye.constroller.OrderHystirxController.default_test_test();
Fallback method: public java.lang.String xyz.xcye.constroller.OrderHystirxController.defaultHandlerMethod();
Hint: Fallback method 'public java.lang.String xyz.xcye.constroller.OrderHystirxController.defaultHandlerMethod()' must return: void or its subclass
```



### 方法二

除了上面的这种解决方法外，还有一种，并且这种方法还能够进行解耦，方法一，在一定程度上，耦合度还是高

因为我们的consumer使用了`OpenFegin`，在controller中，我们都是通过调用该接口，所以我们可以创建一个类，实现使用`@FeignClient`注解标注的接口，然后该类的实现方法，就是consumer调用provider发生服务降级后，进行处理的服务降级方法

1. 接口

   ```java
   @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = PaymentHystrixServiceImpl.class)
   public interface PaymentHystrixService {
   
       @GetMapping("/payment/hystrix/ok/{id}")
       String paymentInfo_OK(@PathVariable("id") Integer id);
       @GetMapping("/payment/hystrix/timeout/{id}")
       String paymentInfo_TimeOut(@PathVariable("id") Integer id);
   }
   ```

   > `fallback`指向该接口的实现类

2. 实现类

   ```java
   @Component
   public class PaymentHystrixServiceImpl implements PaymentHystrixService {
   
       @Override
       public String paymentInfo_OK(Integer id) {
           return "impl ------paymentInfo_OK";
       }
   
       @Override
       public String paymentInfo_TimeOut(Integer id) {
           return "impl----------------paymentInfo_TimeOut";
       }
   }
   ```

   > 一定要将该类，加入到组件中，使用`@Component`注解

3. 主启动类和配置文件中，也是必不可少的

   ```java
   @EnableFeignClients
   @EnableHystrix
   ```

   ```yml
   feign:
     hystrix:
       enabled: true
   ```

4. controller

   ```java
   @DefaultProperties(defaultFallback = "defaultHandlerMethod")
   @RestController
   public class OrderHystirxController {
   
       @Resource
       private PaymentHystrixService paymentHystrixService;
   
       @GetMapping("/consumer/payment/hystrix/ok/{id}")
       String paymentInfo_OK(@PathVariable("id") Integer id) {
           String s = paymentHystrixService.paymentInfo_OK(id);
           return s;
       }
       @HystrixCommand(fallbackMethod = "paymentInfo_TimeOutHandler",commandProperties = {
               @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1000")
       })
       @GetMapping("/consumer/payment/hystrix/timeout/{id}")
       String paymentInfo_TimeOut(@PathVariable("id") Integer id) {
           String s = paymentHystrixService.paymentInfo_TimeOut(id);
           return s;
           //int a = 10 / 0;
           //return "sdf";
       }
   
       String paymentInfo_TimeOutHandler(@PathVariable("id") Integer id) {
           return "80方法调用超时了";
       }
   
       @GetMapping("/consumer/payment/hystrix/default_test")
       public String default_test() {
           int a = 10 /0;
           return "this is 80's default test method";
       }
   
       @HystrixCommand
       @GetMapping("/consumer/payment/hystrix/default_test_test")
       public void default_test_test() {
           int a = 10 /0;
           //return "this is 80's default test method";
           System.out.println("this is 80's default test method");
       }
   
       public String defaultHandlerMethod() {
           return "-----80------defaultHandlerMethod";
       }
   }
   ```

   

可以发现，我们的controller并没有进行什么改变，尽管`paymentInfo_TimeOut`方法，我们使用`@HystrixCommand`注解指定了对应的处理方法，但是最终，也不会走这个指定的方法，最终走的是实现类中的`paymentInfo_TimeOut()`方法，最终会返回`impl----------------paymentInfo_TimeOut`，因为`default_test_test`和`default_test_test`两个方法，并没有在provider中存在，所以并不会在接口实现类中，进行处理

通过这种方法，我们可以实现进一步的解耦操作





![](https://picture.xcye.xyz/image-20220120202257821.png)
