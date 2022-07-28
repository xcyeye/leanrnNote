---
date: 2022/3/24
---

## Failed to start bean ‘documentationPluginsBootstrapper‘； nested exception is java.lang.NullPointerEx

需要在启动类上增加`@EnableWebMvc`

> 原因可能是因为Springfox使用的路径匹配是基于AntPathMatcher的，而Spring Boot 2.6.X使用的是PathPatternMatcher。该注解可以更改匹配规则。你也可以直接修改配置spring.mvc.pathmatch.matching-strategy=ANT_PATH_MATCHER来更改规则



## 解决spring boot 2.6.X不能使用

```yml
spring:
  mvc:
    pathmatch:
      matching-strategy: ant_path_matcher
```



```java
@Bean
public BeanPostProcessor springfoxHandlerProviderBeanPostProcessor() {
    return new BeanPostProcessor() {
        @Override
        @SuppressWarnings({"NullableProblems", "unchecked"})
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
            if (bean instanceof WebMvcRequestHandlerProvider || bean instanceof WebFluxRequestHandlerProvider) {
                try {
                    Field field = ReflectionUtils.findField(bean.getClass(), "handlerMappings");
                    if (null != field) {
                        field.setAccessible(true);
                        List<RequestMappingInfoHandlerMapping> mappings = (List<RequestMappingInfoHandlerMapping>) field.get(bean);
                        mappings.removeIf(e -> null != e.getPatternParser());
                    }
                } catch (IllegalArgumentException | IllegalAccessException e) {
                    throw new IllegalStateException(e);
                }
            }
            return bean;
        }
    };
}
```

主启动类上可以加上`@EnableWebMvc`注解

