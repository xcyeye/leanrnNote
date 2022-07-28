## 解决Spring Cloud Gateway中使用OpenFeign出现的错误

[解决Spring Cloud Gateway中使用OpenFeign出现的错误 | 码农家园 (codenong.com)](https://www.codenong.com/cs106907570/)

如果在gateway中使用openFeign，会报一个错误，也就是HttpMessageConverters这个对象没有导入，在自动配置中，有一个描述

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(HttpMessageConverter.class)
@Conditional(NotReactiveWebApplicationCondition.class)
@AutoConfigureAfter({ GsonAutoConfiguration.class, JacksonAutoConfiguration.class, JsonbAutoConfiguration.class })
@Import({ JacksonHttpMessageConvertersConfiguration.class, GsonHttpMessageConvertersConfiguration.class,
		JsonbHttpMessageConvertersConfiguration.class })
public class HttpMessageConvertersAutoConfiguration {}
```

因为我们gateway是webflux的，所以这里不会将此对象加入到容器中，解决方法就是手动在项目中，导入该bean

```java
@Configuration
public class AuroraGatewayConfig {

    @Bean
    public HttpMessageConverters messageConverters(ObjectProvider<HttpMessageConverter<?>> converters) {
        return new HttpMessageConverters(converters.orderedStream().collect(Collectors.toList()));
    }
}
```

