# OpenFeign Issue

## 服务熔断降级处理

在进行服务降级和熔断处理的时候，处理类上，一定要添加`@Component`注解，否则启动会报错

