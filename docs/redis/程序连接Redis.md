# 程序连接Redis

# Jedis

使用Redis官方推荐的Jedis，在java应用中操作Redis。Jedis几乎涵盖了Redis的所有命令。操作Redis的命令在Jedis中以方法的形式出现。





# 连接池的使用

连接池和mysql的连接池是一样的，一次性创建多个连接，并将他们放在一个容器中，每一次从里面进行取，如果使用完之后，调用`close()`方法可以这个连接放回容器中



## 导包

一定呀导入这几个包，否则会报一个错误

`java.lang.NoClassDefFoundError: org/slf4j/LoggerFactory`

![](https://picture.xcye.xyz/image-20210310211417577.png?x-oss-process=style/pictureProcess1)

连接池包

![](https://picture.xcye.xyz/image-20210310211505942.png?x-oss-process=style/pictureProcess1)

如果没有指定配置文件的话，那么默认的连接个数为

```java
private int maxTotal = 8;
private int maxIdle = 8;
private int minIdle = 0;
```



如果使用for循环进行获取

```java
for (int i = 0; i < 15; i++) {
   Jedis jedis = pool.getResource();
   System.out.println(i+"--->"+jedis);
}
```

那么到i=7时，就会一直等待，不会再打印，可以在每一次使用完成之后，就将其`close()`放回

返回一个

```
0--->redis.clients.jedis.Jedis@21a947fe
1--->redis.clients.jedis.Jedis@10b48321
2--->redis.clients.jedis.Jedis@6b67034
3--->redis.clients.jedis.Jedis@16267862
4--->redis.clients.jedis.Jedis@453da22c
5--->redis.clients.jedis.Jedis@71248c21
获取的是同一个
6--->redis.clients.jedis.Jedis@442675e1
7--->redis.clients.jedis.Jedis@442675e1

8--->redis.clients.jedis.Jedis@6b0c2d26
```



自定义连接池的个数

```java
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(5);
config.setMaxIdle(50);
JedisPool pool = new JedisPool(config,"localhost",6389);
```





## 连接池工具类

可以像mysql那样，创建一个连接池工具类，这样使用的时候，会更加的方便

```java
package com.chu.util;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 *此类是Redis连接池的工具类
 */
public class RedisUtil {
    private static JedisPool pool;
    static {
        InputStream stream = RedisUtil.class.getClassLoader().getResourceAsStream("com/chu/redis/jedis.properties");
        Properties pro = new Properties();
        try {
            pro.load(stream);
        } catch (IOException e) {
            e.printStackTrace();
        }
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));
        //使用config初始化连接池
        pool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));
    }
    public static Jedis getJedis() {
        return pool.getResource();
    }
}
```



### 工具类的使用

```java
package com.chu.redis;

import com.chu.util.RedisUtil;
import redis.clients.jedis.Jedis;

import java.util.Set;

public class Demon4 {
    public static void main (String[] args) {
        //测试连接池工具类
        Jedis jedis = RedisUtil.getJedis();
        Set<String> keys = jedis.keys("*");
        for (String key : keys) {
            System.out.println(key);
        }
    }
}
```



## 使用Redis作为缓存

![](https://picture.xcye.xyz/image-20210311130255828.png?x-oss-process=style/pictureProcess1)

程序代码

```java
public String find () {
        //此方法用于Redis的缓存功能
        //web层调用此方法，第一次的时候，Redis中都没有数据，需要进行判断
        //之后每一次都向Redis请求数据就行，这种使用Redis作为缓存的方法适用于数据库中的数据不经常发生改变的情况
        //判断Redis中是否有数据
        Jedis jedis = RedisUtil.getJedis();
        String select = jedis.select(1);
        System.out.println(select);
        if (jedis.get("province") == null || jedis.get("province").length() == 0) {
            //Redis中的province值为null，或者长度为空，代表没有数据
            ProvinceDao province = new ProvinceDaoImpl();
            List<Province> list = province.findAll();
            //将list转化为json，并且存入到Redis中
            ObjectMapper mapper = new ObjectMapper();
            String json = null;
            try {
                json = mapper.writeValueAsString(list);
            } catch (JsonProcessingException e) {
                e.printStackTrace();
            }
            jedis.set("province",json);
            System.out.println("redis中没有数据 存入的数据为: "+jedis.get("province"));
            //返回Redis中的数据
            return jedis.get("province");
        }
        //程序能运行到这里，说明Redis中存在数据
        System.out.println("Redis中存在数据，为: "+jedis.get("province"));
        return jedis.get("province");
    }
```



因为第一次启动的时候(`启动服务器之后，最先访问的那一个，Redis中没有设置这个key`)，那么需要将mysql中的数据保存至Redis中，这样下次访问的时候，就可以不用再去想mysql请求数据，可以在很大程度上减少访问的时间





## 测试Redis查询相同的数据和mysql的时间比较

```java
package com.chu.web.servlet;

import com.chu.domain.Province;
import com.chu.service.ProvinceService;
import com.chu.service.impl.ProvinceServiceImpl;
import com.fasterxml.jackson.databind.ObjectMapper;

import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;
import java.util.Date;
import java.util.List;

@WebServlet("/ServletRedis2")
public class ServletRedis2 extends HttpServlet {
    @Override
    protected void doGet (HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }

    @Override
    protected void doPost (HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //此类用于测试，向mysql中请求数据和Redis中请求数据的时间对比
        //设置响应的编码方式
        response.setContentType("text/json;charset=utf-8");

        ObjectMapper mapper = new ObjectMapper();
        //调用service层方法返回一个集合
        ProvinceService service1 = new ProvinceServiceImpl();

        //向mysql中请求数据
        long mysql_start = new Date().getTime();
        List<Province> list = service1.findAll();
        String mysql_json = mapper.writeValueAsString(list);
        System.out.println("mysql: "+mysql_json);
        long mysql_end = new Date().getTime();
        System.out.println("mysql花费时间: "+(mysql_end-mysql_start));
        System.out.println("------------------------\n\n\n\n\n\n\n\n\n");
        //Redis测试
        long redis_start = new Date().getTime();
        String redis_json = service1.find();
        long redis_end = new Date().getTime();
        System.out.println("redis: "+redis_json);
        System.out.println("redis花费时间: "+(redis_end-redis_start));
        System.out.println("相差: "+((mysql_end-mysql_start)-(redis_end-redis_start)));
    }
}
```



Redis完胜

