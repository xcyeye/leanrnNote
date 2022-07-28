---
date: 2022/4/2
---

# spring cloud stream 3.2学习记录

```yml
spring:
    cloud:
        stream:
          function:
            definition: testFunc
          bindings:
            testFunc-out-0:
              destination: aurora-stream-test
```

在配置的时候，无论存在一个bean还是存在多个bean，我们都需要配置`function.definition`

![image-20220408225931323](https://picture.xcye.xyz/image-20220408225931323.png)

如果没有生产者没有设置stream.bindings.xxx.destination，但是设置了rabbitmq.bindings.xxx.producer的相关属性的时候，那么交换机的名称就是方法名称

如果我们指定`stream.bindings.xxx.destination`，那么这个值就是交换机的名称



其中stream.bindings.xxx.就是信道名称

## streamBridge使用

```java
@Autowired
private StreamBridge streamBridge;

public String test() {
    long l = GenerateInfoUtils.generateUid(1, 2);
    boolean send = streamBridge.send("provider1-out-0", l);
    return l + "发送: " + send;
}
```

这个send方法有多个重载，其中第一个参数，可以是绑定的名称，也可以是交换机名称(如果`spring.cloud.stream.bindings.xxx-out-0.destination`没有设置的时候，会将此名称作为交换机名称)

我们可以在send方法中发送Object类型的数据，如果想要正常使用，我们还需要在配置文件中，加上一些配置，可以通过`spring.cloud.stream.bindings.xxx-out-0`和`spring.cloud.stream.rabbit.bindings.<channelName>.producer.`进行设置



## topic交换机

```java
@GetMapping("/msg3")
    public String test3() {
        boolean send = streamBridge.send("topicPro-out-0", "sdfsdf");
        return "";
    }
```



```
spring.cloud.bindings:
        topicPro-out-0:
          destination: nimade
```



消费者，这里设置四个消费者

```java
@Bean
    public Consumer<String> consumer1() {
        return msg -> {
            System.out.println("交换机1号: " + msg);
        };
    }


    @Bean
    public Consumer<String> consumer3() {
        return msg -> {
            System.out.println("交换机3号: " + msg);
        };
    }

    @Bean
    public Consumer<String> consumer4() {
        return msg -> {
            System.out.println("交换机4号: " + msg);
        };
    }
```

```yaml
spring:
	  cloud:
    stream:
      bindings: # 服务的整合处理
        consumer1-in-0:
          destination: nimade
        consumer3-in-0:
          destination: nimade
        consumer4-in-0:
          destination: nimade
      function:
        definition: consumer1;consumer3;consumer4
```

如果有多个消费者的话，不管是和哪种交换机进行绑定，都一定要设置`spring.cloud.function.definition`，值是方法名，如果存在多个，使用`;`进行分隔开

![image-20220409002413733](https://picture.xcye.xyz/image-20220409002413733.png)

![image-20220409002359560](https://picture.xcye.xyz/image-20220409002359560.png)

## fanout交换机的绑定

生产者

```java
@GetMapping("/msg4")
public String test4() {
    streamBridge.send("fanoutPro-out-0",System.currentTimeMillis());
    return System.currentTimeMillis() + "";
}
```

```yml
spring:
	cloud:
		stream:
			bindings: # 服务的整合处理
				fanoutPro-out-0:
					destination: fanout-ex

	cloud:
    stream:
      rabbit:
        bindings:
          fanoutPro-out-0:
            producer:
              exchangeType: fanout
```



消费者

```java
@Bean
public Consumer<String> consumer3() {
    return msg -> {
        System.out.println("交换机3号: " + msg);
    };
}

@Bean
public Consumer<String> consumer4() {
    return msg -> {
        System.out.println("交换机4号: " + msg);
    };
}
```

