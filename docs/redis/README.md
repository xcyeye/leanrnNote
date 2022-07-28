# 什么是redis

Redis是一个数据库软件，DBMS，



## 数据库时代的发展

### 单机数据库时代，一个应用，就创建一个数据库实例

### 缓存，水平切分时代：

>  比如在面对一个购物系统的时候，会出现上亿条的订单，物流等等记录，每一个数据库的空间都是有限的，不能够存放这么多张表，这个时候，就可以使用切分的方式

订单表，有很多条，就单独创建一个数据库用来存放这个订单的数据

其他订单量小的表，可以几张表存放在一个数据库中，到时候需要使用的时候，就直接连接到那个数据库就行

###  读写分离时代

> 像京东，淘宝这样，每天，每一个时间，都会有很多的订单查询，修改等等的操作，这个时候，水平切分的方式已经不能够满足用户的需求，这个时候就需要使用到数据库的集群，
>
> 读写分离的方式，也是，所有的读订单的操作，都是那个或几个数据库中进行，所有的修改操作都去另外的几个数据库中进行，等等，所有的数据库是同步的，也就是某个数据库中的表发生了改变，那么就会立马将这个记录通过到其他的数据库中(有这条记录的表)进行修改，这种集群的方式就可以使得在拥有很大并发量的情况下，用户能够正常的进行操作

### 分库分表时代

> 对于很多很大的数据，可能读写分离的方式又不能满足要求了，就比如某个订单拥有3亿多条记录，读写分离的方式可能还在存在慢的情况，这个时候就可以使用分库分表的方式，也就是将所有的记录进行拆分，这一天，这一个月，这一年，某个时刻，都用来创建数据库，可能一年就会创建12个数据库(按月)或者365（按天），这个时候，就可以将一个月或者一天的记录放在这个数据库中，这样用户进行操作的时候，就可以去那一个月或者天当中的库中查找
>
> 但是可能我们需要自己写一个程序，根据用户的不能操作去找不同的数据库



`上面的技术并没有消失，我们可能都还会使用，需要根据实际的情况决定使用哪种方式`

### ` 上面的方式，都是存在于关系型数据库的情况`

所以就有了非关系型数据库的发展，从`底层该表了库的存储结构`，不再使用表的方式存储数据，而是使用聚合数据结构的方式进行存储

# Redis

Redis是使用`键值对`方式(k value)

是使用C语言编写的，开源，`基于内存运行的`

## 基于内存

>  **`一定要记住基于内存运行`**

为什么要基于内存？

> 进行操作的时候，Redis是将数据写入内存，内存的运行速率是很大的，所以这就可以操作读写的速度非常快



Redis中的数据大多数情况下，都是在内存中，

缺陷就是，内存的容量小，Redis追求的是速度的极致，所以不能存放太多数据的情况，并且存放在Redis中的数据，最好是那种，经常使用，经常读写的数据

对于需要存放很大的数据记录，就不应该使用Redis进行存储，可以使用其他的非关系型数据库，像(mongDB)





# 安装

一般是安装在Linux上，但是也可以安装在Windows上

安装步骤

1. 下载`zip`包，解压

2. cmd进入到这个解压包

3. 输入命令`redis-server redis.windows.conf`

4. 开启服务，输入命令`redis-server --service-install redis.windows.conf`

5. 手动开启服务

    ![](https://picture.xcye.xyz/image-20210305213204405.png?x-oss-process=style/pictureProcess1)

> 常用命令
>
> ```
> 卸载服务：redis-server --service-uninstall
> 开启服务：redis-server --service-start
> 停止服务：redis-server --service-stop
> ```



更改参数

> 服务器端：`redis.windows-service.conf`改端口：`port 6379`
>
> 客户端：`redis.windows.conf`



## windows启动Redis

因为Redis的服务器和客户端都是使用电脑，多以需要开两个cmd窗口，一个作为服务器，一个作为客户端



启动都需要进入到Redis的目录中，

- 第一种

启动服务器，命令

>  `redis-server.exe redis.windows.conf`

启动客户端

>  `redis-cli.exe -h 127.0.0.1 -p 6379`



- 第二种方式

    启动服务器的方式一样

    客户端进入Redis目录，输入

    `redis-cli`,也就是打开此目录中的`redis-cli.exe`文件

    此客户端是Redis自带的客户端，只要安装了Redis就有这个客户端



默认连接的是本机上的Redis

也可以通过主机连接其他服务器上的Redis服务

> `redis-cli.exe -h IP地址 -p 端口`



## windows退出客户端



> `exit`
>
> `quit`





## Linux安装Redis

1. 运行`wget http:``//download.redis.io/releases/redis-5.0.7.tar.gz`

2. 解压`tar -zvxf redis-5.0.7.tar.gz`

3. 进入到Redis解压目录中 执行`make`命令进行编译，这里如果发生意外

    https://www.bilibili.com/video/BV1Uz4y1X72A?p=5

4. 编译完成之后，就执行`make install`，Redis是使用C语言写的，需要进行编译



### 启动

服务器启动

> 后台启动`redis-service redis.conf`(需要将其设置为守护线程)
>
> 前台启动`redis-service`
>
> 启动时，可以指定配置文件进行启动：`redis-server redis.conf &`
>
> 最好是使用后台启动的方式，如果使用配置文件进行启动，并且没有设置守护线程，那么还是前台启动

查看端口号

> `ps -ef|grep redis`

![](https://picture.xcye.xyz/image-20210305232021457.png?x-oss-process=style/pictureProcess1)

使用命令

> `cd src`
>
> `ls`可以看到编译后的文件

![](https://picture.xcye.xyz/image-20210305232324347.png?x-oss-process=style/pictureProcess1)

绿色的文件都是可执行的文件



- `make install`命令的作用就是，将src目录下编译的可执行文件拷贝到`usr/local/bin`目录中，这样在任意位置都可以执行这些文件

![](https://picture.xcye.xyz/image-20210305232727594.png?x-oss-process=style/pictureProcess1)



### 关闭Redis

- 暴力方式

    直接断开连接

- `ps -ef|grep redis`查看端口，然后结束任务

- `redis-cli shutdown`推荐，暴力的方式，数据可能会丢失

- 如果是前台启动，则输入`Ctrl + C`直接关闭

### 客户端连接

使用命令

>  `redis-cli`，需要先打开服务器，这种方式默认连接的是本机上的Redis

连接某个IP中的Redis

> `redis-cli -h IP地址 -p 端口(空格)`



### 客户端退出

`quit or exit`



# Redis基本知识



## 测试Redis的服务性能

> `redis-benchmark`

![](https://picture.xcye.xyz/image-20210306091624123.png?x-oss-process=style/pictureProcess1)

![](https://picture.xcye.xyz/image-20210306091638509.png?x-oss-process=style/pictureProcess1)

```ssh
====== SET ======                                                    
  100000 requests completed in 1.01 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
  host configuration "save": 3600 1 300 100 60 10000
  host configuration "appendonly": no
  multi-thread: no
```

## ping，命令

```
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> 

```

此命令可以查看Redis服务的开启状态，是否可用，会面临着服务器和客户端不在同一台电脑上，这个时候，就可以使用这个命令查看Redis的服务时候可用

`正常返回pong`

## 查看redis服务器的统计信息：info

语法：`info [section]`

作用：以一种易于解释且易于阅读的格式，返回关于 Redis 服务器的各种信息和统计数值。section 用来返回指定部分的统计信息。 section的值：server , clients ，memory等等。不加section 返回全部统计信息

返回值：指定section的统计信息或全部信息

- 查看server的统计信息

    ![](https://picture.xcye.xyz/image-20210306092233833.png?x-oss-process=style/pictureProcess1)

- 查看全部的统计信息

    ![](https://picture.xcye.xyz/image-20210306092254854.png?x-oss-process=style/pictureProcess1)

    - 服务器端的信息

        ```
        Server
        redis_version:6.2.1    版本
        redis_git_sha1:00000000
        redis_git_dirty:0
        redis_build_id:a99f87a74fdf9f96
        redis_mode:standalone
        os:Linux 5.4.0-65-generic x86_64
        arch_bits:64
        multiplexing_api:epoll
        atomicvar_api:c11-builtin
        gcc_version:9.3.0
        process_id:118619
        process_supervised:no
        run_id:e675ab60db6eb126d15dc83c94114c86b5d89c8c
        tcp_port:6379
        server_time_usec:1614993780096472
        uptime_in_seconds:35350
        uptime_in_days:0
        hz:10
        configured_hz:10
        lru_clock:4381044
        executable:/redis/redis-server
        config_file:
        io_threads_active:0
        ```

    - 客户端

        ```
        connected_clients:1   目前连接服务器的个数
        cluster_connections:0
        maxclients:10000
        client_recent_max_input_buffer:16
        client_recent_max_output_buffer:0
        blocked_clients:0
        tracking_clients:0
        clients_in_timeout_table:0
        
        ```

    - 内存

        ```
        # Memory
        used_memory:2323912
        used_memory_human:2.22M
        used_memory_rss:7839744
        used_memory_rss_human:7.48M
        used_memory_peak:5858176
        used_memory_peak_human:5.59M
        used_memory_peak_perc:39.67%
        used_memory_overhead:830480
        used_memory_startup:809760
        used_memory_dataset:1493432
        used_memory_dataset_perc:98.63%
        allocator_allocated:2763256
        allocator_active:3141632
        allocator_resident:5562368
        total_system_memory:1975848960
        total_system_memory_human:1.84G
        used_memory_lua:37888
        used_memory_lua_human:37.00K
        used_memory_scripts:0
        used_memory_scripts_human:0B
        number_of_cached_scripts:0
        maxmemory:0
        maxmemory_human:0B
        maxmemory_policy:noeviction
        allocator_frag_ratio:1.14
        allocator_frag_bytes:378376
        allocator_rss_ratio:1.77
        allocator_rss_bytes:2420736
        rss_overhead_ratio:1.41
        rss_overhead_bytes:2277376
        mem_fragmentation_ratio:3.44
        mem_fragmentation_bytes:5558600
        mem_not_counted_for_evict:0
        mem_replication_backlog:0
        mem_clients_slaves:0
        mem_clients_normal:20496
        mem_aof_buffer:0
        mem_allocator:jemalloc-5.1.0
        active_defrag_running:0
        lazyfree_pending_objects:0
        lazyfreed_objects:0
        ```

## Redis的16个库

Redis默认使用16个库，从0到15。 对数据库个数的修改，在`redis.conf`文件中databases 16，理论上可以配置无限多个。

Redis中的库，不像mysql中的那样，可以自己修改，更改数据库的名字，但是这样也就导致了，mysql中每一个库占的内存可能达到`几百M`，因为每一个库中都有处理这一个库的程序

而Redis每一个库，占的内存特别的小，初始化的时候，没存数据，只达到`几KB`，非常小

 

- 修改初始化的库个数

    文件`redis.conf`

    ![](https://picture.xcye.xyz/image-20210306093112737.png?x-oss-process=style/pictureProcess1)

    Redis的库和关系型数据库中的数据库实例类似，但又有一些不同，比如redis中各个库`不能自定义命名`，只能用`序号`表示，也就是从0开始，redis中各个库不是完全独立的，使用时最好一个应用使用一个redis实例，不建议一个redis实例中保存多个应用的数据。Redis实例本身所占存储空间其实是非常小的，因此不会造成存储空间的浪费。



### 切换库

>  切换库命令：`select db`

默认使用第0个，如果要使用其他数据库，命令是 `select index`

![](https://picture.xcye.xyz/image-20210306093307660.png?x-oss-process=style/pictureProcess1)

如果超过库的总个数，会报下标越界错误

```
127.0.0.1:6379[15]> select 16
(error) ERR DB index is out of range
```



### 查看当前数据库中key的数目：dbsize

> 语法：`dbsize`

作用：返回当前数据库的 key 的数量。

返回值：数字，key的数量

![](https://picture.xcye.xyz/wps1.jpg?x-oss-process=style/pictureProcess1) 

### 查看当前数据库中有哪些key：`keys *`

![](https://picture.xcye.xyz/wps2.jpg?x-oss-process=style/pictureProcess1) 

### 清空当前库：`flushdb`

![](https://picture.xcye.xyz/wps3.jpg?x-oss-process=style/pictureProcess1)

清空当前数据库中国的key 

### 清空所有数据库：`flushall`

![](https://picture.xcye.xyz/wps4.jpg?x-oss-process=style/pictureProcess1) 

这也体现出redis中的库并不是完全无关的。

`执行这个命令的话，那么当前Redis 中所有的库中的信息都会被删除`



### config get * 获得redis的所有配置值

> 语法：`config get parameter`

作用：获取运行中Redis服务器的配置参数， 获取全部配置可以使用*。

参数信息来自redis.conf 文件的内容。

 

- 例1：获取数据库个数 `config get databases`

![](https://picture.xcye.xyz/wps5.jpg?x-oss-process=style/pictureProcess1) 

- 例2：获取端口号`config get port`

![](https://picture.xcye.xyz/wps6.jpg?x-oss-process=style/pictureProcess1) 

手册地址：

redis英文版命令大全：https://redis.io/commands

redis中文版命令大全：http://redisdoc.com/
