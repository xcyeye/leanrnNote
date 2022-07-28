---
date: 2022/1/18 16:01
tag: [spring cloud,openFeign,feign,spring boot,微服务]
---

# OpenFeign服务接口调用

https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/#spring-cloud-openfeign

Feign是一个声明式WebService客户端，使用Feign能让编写Web Service客户端更加简单。

它的使用方法是定义一个服务接口然后在上面添加注解。

Feign也支持可拔插式的编码器和解码器，Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters，Feign可以与Eureka和Ribbon组合使用以支持负载均衡

我们通过查看该依赖，也能够看到，在`OpenFeign`依赖中，存在`Ribbon`依赖

![](https://picture.xcye.xyz/image-20220118160806649.png)


Feign旨在使编写Java Http客户端变得更容易。
前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解,现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

Feign集成了Ribbon
利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用



## Feign和OpenFeign的区别

| Feign                                                        | OpenFeign                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端<br/>Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务。Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务 | OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等。OpenFeign的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务。 |
| `spring-cloud-starter-feign`                                 | `spring-cloud-starter-openfeign`                             |

> openFeign是服务端组件，所以我们需要在消费者上使用



## 使用

这里有两个服务，service1和service2，其中service2作为消费者

service1的接口为

```java
@RequestMapping("/message/messageLog")
@RestController
public class MessageLogController {
    @Autowired
    private MessageLogService messageLogService;

    @ResponseResult
    @PostMapping("/insert")
    public ModifyResult insertMessageLog(@Validated({Insert.class,Default.class}) MessageLogDO messageLogDO) throws BindException {
        return messageLogService.insertMessageLog(messageLogDO);
    }

    @ResponseResult
    @PutMapping("/update")
    public ModifyResult updateMessageLog(@Validated(Update.class) MessageLogDO messageLogDO) throws BindException {
        return messageLogService.updateMessageLog(messageLogDO);
    }

    @ResponseResult
    @DeleteMapping("/delete/{uid}")
    public ModifyResult deleteMessageLog(@PathVariable("uid") long uid) {
        return messageLogService.deleteMessageLog(uid);
    }

    @ResponseResult
    @GetMapping("/queryAll")
    public List<MessageLogVO> queryAllMessageLog(MessageLogDO messageLogDO, PaginationDTO paginationDTO) throws InstantiationException, IllegalAccessException {
        return messageLogService.queryAllMessageLog(messageLogDO,paginationDTO);
    }

    @ResponseResult
    @GetMapping("/queryByUid/{uid}")
    public MessageLogDO queryMessageLogByUid(@PathVariable("uid") long uid) throws InstantiationException, IllegalAccessException {
        return messageLogService.queryByUid(uid);
    }
}
```





service2消费者的使用步骤

1. 创建一个feign接口

    ```java
    @Component
    @FeignClient(value = "aurora-message",contextId = "aurora-message-messageLog")
    public interface MessageLogFeignService {
    
        @PostMapping("/message/messageLog/insert")
        ModifyResult insertMessageLog(@SpringQueryMap MessageLogDO messageLogDO) throws BindException;
    
    
        @PutMapping("/message/messageLog/update")
        ModifyResult updateMessageLog(@SpringQueryMap MessageLogDO messageLogDO) throws BindException;
    
        @GetMapping("/message/messageLog/queryByUid/{uid}")
        MessageLogDO queryMessageLogByUid(@PathVariable long uid) throws InstantiationException, IllegalAccessException;
    }
    ```

    其中`@FeignClient(value = "aurora-message",contextId = "aurora-message-messageLog")`的name就是service2服务的名称，contextId是一个唯一id，接口中我们的接口地址需要和service1中的地址对应，并且在feign接口中，不能使用`RequestMapping`统一定义接口地址前缀

    如果service1接口中的方法参数是一个对象的话，在feign接口中，需要加上`@SpringQueryMap`
