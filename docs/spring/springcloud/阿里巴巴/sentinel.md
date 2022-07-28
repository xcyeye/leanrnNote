## gateway限流

官方文档

https://github.com/alibaba/spring-cloud-alibaba/wiki/Sentinel#spring-cloud-gateway-%E6%94%AF%E6%8C%81



自定义异常消息

![image-20220518152829407](https://picture.xcye.xyz/image-20220518152829407.png)

![image-20220519122602133](https://picture.xcye.xyz/image-20220519122602133.png)

一定需要加上这个依赖

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



