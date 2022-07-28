# Redis 集群

#  主从复制

什么是主从复制，主库负责写操作，从库负责读操作，主库写入数据，会将这些数据复制到从库中进行



>  `特征：主少从多，主写从读，读写分离，主写同步复制到从`
>
> 一般主库进行写入的次数比较少，相对于从库来说，很多的时候，都是进行读取的操作，



# 搭建一主二从集群

正常情况下，搭建一主二从集群，需要在在3个服务器中进行，一个服务器安装一个Redis，但是个人使用，用作测试，可以在一个服务器中启动三次服务，但是需要保证每一次启动服务器的端口是不相同的，这样就可以保证同一个服务器中的Redis能够独立运行

所以每一次启动的时候，都需要更改配置文件的设置，并且使用配置文件进行启动



>  `小贴士：因为配置文件的名字可以随便命名，所以可以这样 6379redis.conf,6380redis.conf....`



## 修改配置文件

因为需要启动三次，所以需要修改配置文件中的端口，这样能够使他们都能够独立的运行

    bind 127.0.0.1  为了连接方便，在公司中，一般使用IP进行连接
    port 6379       端口必须要进行更改
    pidfile /var/run/redis_6379.pid  推荐更改端口之后，就改一些这个配置
    logfile "6379.log"               将日志打印开启，为了方便查看每一个库的执行
    dbfilename dump6379.rdb          这个用于每一个库的恢复备份数据，如果三个库中的数据，全部在同一个文件中，那么就特别不好看，不知道哪些数据时那个库中的


Linux中复制文件的命令`cp 文件名 新文件名`



进行连接`redis-server redis6379.conf &....`



##  启动主机和从机

- 查看三台Redis服务在集群中的角色

    默认的角色都是master，这是一种可读可写的角色

    使用命令` info replication`可以查看

    ```
    127.0.0.1:6379>  info replication
    # Replication
    role:master   角色
    connected_slaves:0  连接的用户数
    master_failover_state:no-failover
    master_replid:7bec8d4ea6fe997bf940ea5b089ec6d66b7739cd
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:0
    second_repl_offset:-1
    repl_backlog_active:0
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:0
    repl_backlog_histlen:0
    ```

- `所有的Redis服务默认情况下都是主机，他们可以进行读和写，他们默认都是没有从机的`



- 最好先测试三个服务之间有没有关系，可以在6379主机上添加一个数据，在在其他两台服务上查看是否存在`set k1 v1`，因为没有设置主从关系，他们不存在



- 设置主从关系，使用命令`slaveof 127.0.0.1 6379` 

    slaveof是从属的关系

    > 语法`slaveof 主机IP 主机端口`



- 再次查看服务的角色

    - 在丛机上进行查看

    ```
    127.0.0.1:6380> info replication
    # Replication
    role:slave    角色切换为丛机
    master_host:127.0.0.1  连接到的主机IP
    master_port:6379  主机端口
    master_link_status:up  连接状态
    master_last_io_seconds_ago:8
    master_sync_in_progress:0
    slave_repl_offset:70
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_failover_state:no-failover
    master_replid:8c88cab106493226833dead0b6d3f5a006b18866
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:70
    second_repl_offset:-1
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:1
    repl_backlog_histlen:70
    ```

    

    - 主机查看

        ```
        127.0.0.1:6379> info replication
        # Replication
        role:master   角色为主机
        connected_slaves:2  连接到主机的丛机个数
        slave0:ip=127.0.0.1,port=6380,state=online,offset=224,lag=1  从机信息
        slave1:ip=127.0.0.1,port=6381,state=online,offset=224,lag=0  从机信息
        master_failover_state:no-failover
        master_replid:8c88cab106493226833dead0b6d3f5a006b18866
        master_replid2:0000000000000000000000000000000000000000
        master_repl_offset:224
        second_repl_offset:-1
        repl_backlog_active:1
        repl_backlog_size:1048576
        repl_backlog_first_byte_offset:1
        repl_backlog_histlen:224
        ```

    `因为方才在6379上设置了一个键k1，现在在6380机上查看时，主机中的数据就会自动服务到丛机中`

![](https://picture.xcye.xyz/image-20210309223552465.png?x-oss-process=style/pictureProcess1)







- 全量复制和增量复制的区别

    > `全量复制`：一旦主从关系确定，会自动把主库上已有的数据同步复制到从库。 
    >      在6380和6381上执行：keys *
    > `增量复制`：主库写数据会自动同步到从库。
    >      在6379上执行：set k2 v2
    >      在6380和6381上执行：keys *



- 测试从机是否可以写和读操作

    - 丛机

        ```
        127.0.0.1:6380> set k2 v2
        (error) READONLY You can't write against a read only replica.
        127.0.0.1:6380> get k1
        "v1"
        ```

        丛机不可以进行写操作，会报错，但是可以读

    - 主机

        ```
        127.0.0.1:6379> get k1
        "v1"
        127.0.0.1:6379> set k2 v2
        OK
        ```

        主机既可以进行写操作，又可以进行读操作，但是一般不建议主机进行读操作

    `这个就是主写从读，读写分流`



## 主机宕机

如果主机发生了宕机行为(`shutdown服务`)，那么丛机会处于等待状态，连接状态变为`down`，如果主机重新开启，丛机会自动进行连接

![](https://picture.xcye.xyz/image-20210309224934156.png?x-oss-process=style/pictureProcess1)

丛机情况

![](https://picture.xcye.xyz/image-20210309225003740.png?x-oss-process=style/pictureProcess1)

但是尽管主机处于宕机状态，丛机还是可以进行读操作，只是丛机中的数据不会更新，因为主机宕机，没有数据的复制



### 主机开启

如果主机宕机之后，又重新启动，那么从机会立即进行连接，将连接状态改为`up`，主机会将数据复制到丛机中



## 丛机宕机



如果丛机发生宕机行为，那么主机会立即失去一个丛机，主机的角色中的丛机连接也会少一个，



### 丛机开启

如果丛机从发生宕机后又重新开启服务，那么丛机不会立即再次变为丛机，而是成为主机，这个服务会失去丛机的角色

如果还想将这个服务设置为丛机，那么需要再次运行命令`slaveof IP port`





## 丛机上位

如果遇到主机宕机，而且这种情况很严重，已经很难再次维护运行，所以这个时候，就可以让丛机中性能比较的来当主机，如果这样设置的话，丛机中原来有一些从原先主机中复制过来的数据，这些数据从机上位后就是自己的了



### `slaveof no one`

在性能比较好的那个丛机输入命令`slaveof no one`，就是让这个Redis服务，不从属于谁，所以这个Redis服务就从丛机变为主机

```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:1550
master_link_down_since_seconds:1283
slave_priority:100
slave_read_only:1
connected_slaves:0
master_failover_state:no-failover
master_replid:8c88cab106493226833dead0b6d3f5a006b18866
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1550
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1550
127.0.0.1:6380> slaveof no one
OK
上面这个Redis服务还是丛机
127.0.0.1:6380> info replication
# Replication
role:master  变为主机
connected_slaves:0
master_failover_state:no-failover
master_replid:4decd8d9573264f63a7ca442a271022e89ada9b3
master_replid2:8c88cab106493226833dead0b6d3f5a006b18866
master_repl_offset:1550
second_repl_offset:1551
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1550
```



然后在另一个Redis服务中重新设置其从属关系(原来主机的另一个丛机)



还是使用命令`slaveof 新主机IP port`

```
127.0.0.1:6381> slaveof 127.0.0.1 6380  重新射中从属关系
OK

127.0.0.1:6381> keys *
1) "k1"
2) "k2"
3) "k3"
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:1644
slave_priority:100
slave_read_only:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4decd8d9573264f63a7ca442a271022e89ada9b3
master_replid2:8c88cab106493226833dead0b6d3f5a006b18866
master_repl_offset:1644
second_repl_offset:1551
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:15
repl_backlog_histlen:1630
127.0.0.1:6381> 
```





## 原来的主机从宕机到启动

如果原来的主机从宕机状态恢复到启动，因为他原来的丛机已经更改了从属关系，所以尽管原来有丛机，但是现在不会有

他现在可以有两个选择，一个是继续当主机，那么正在运行的那个有丛机的主机就要取消主机上位，

这个从宕机到启动的主机还可以当从机，可以是6380的丛机，也可以是6381的丛机，可以进行嵌套使用，但是对于6381端口的服务来说，他就有两个角色，对于6380主机来说，6381服务是一个丛机，但是对于6379服务来说，6381服务是一个主机，6379中的数据会从6381机中进行复制

- 6379服务设置为6381机的丛机

    ```
    127.0.0.1:6379> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6381
    master_link_status:up
    master_last_io_seconds_ago:3
    master_sync_in_progress:0
    slave_repl_offset:3380
    slave_priority:100
    slave_read_only:1
    connected_slaves:0
    master_failover_state:no-failover
    master_replid:4decd8d9573264f63a7ca442a271022e89ada9b3
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:3380
    second_repl_offset:-1
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:3367
    repl_backlog_histlen:14
    127.0.0.1:6379> keys *
    1) "k3"
    2) "k1"
    3) "k2"
    ```

    



- 6380主机

    ```
    127.0.0.1:6380> info replication
    # Replication
    role:master
    connected_slaves:1
    slave0:ip=127.0.0.1,port=6381,state=online,offset=3422,lag=1
    master_failover_state:no-failover
    master_replid:4decd8d9573264f63a7ca442a271022e89ada9b3
    master_replid2:8c88cab106493226833dead0b6d3f5a006b18866
    master_repl_offset:3422
    second_repl_offset:1551
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:1
    repl_backlog_histlen:3422
    ```



- 6381机角色信息

    ```
    127.0.0.1:6381> info replication
    # Replication
    role:slave
    master_host:127.0.0.1
    master_port:6380
    master_link_status:up
    master_last_io_seconds_ago:7
    master_sync_in_progress:0
    slave_repl_offset:3492
    slave_priority:100
    slave_read_only:1
    connected_slaves:1
    下面是其丛机的信息
    slave0:ip=127.0.0.1,port=6379,state=online,offset=3492,lag=0
    master_failover_state:no-failover
    master_replid:4decd8d9573264f63a7ca442a271022e89ada9b3
    master_replid2:8c88cab106493226833dead0b6d3f5a006b18866
    master_repl_offset:3492
    second_repl_offset:1551
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_off
    ```

    



他们之间的关系为：

> 6380主机----> 6381(6380的丛机) ---->6379(6381的丛机)
>
> `虽然6380是6379的原则上的父主机，但是在6380的角色信息中并没有看到6379是6380的丛机信息`
>
> 在6381中可以看到



### 嵌套模式的丛机还可以写？

那么既然6381机是6379的主机，那么他是否具有写入的权限？

`只要其角色上带有丛机的信息，就都不能在进行写入操作，尽管这个Redis服务是另一个Redis的主机，也是不可以进行写入操作的`





# Redis哨兵模式

从机上位的自动版。Redis提供了哨兵的命令，哨兵命令是一个独立的进程，哨兵通过发送命令，来监控主从服务器的运行状态，如果检测到master故障了根据投票数自动将某一个slave转换master，然后通过消息订阅模式通知其它slave，让它们切换主机。然而，一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多哨兵进行监控。



在开发中，可能会遇到这种问题，当搭建一个Redis集群的时候，我们不知道主库什么时候发生意外，那么我们就不能及时的让从库上位，为了解决这个问题，我们就需要使用哨兵模式

哨兵模式就相当于是一个哨兵，时刻坚守在自己的岗位上

为主库设置一个哨兵(`每一个库IP都可以设置一个ip，主库和从库嵌套`)，设置的这个哨兵会一直监视主库的状态，包括连接到主库的从库信息，那么当主库发生意外的时候，哨兵就会立马作出反应(`哨兵会存在几秒等的延迟`)，哨兵就会对从库进行投票算法(`一般还是性能比较好的，得到的票数多`)

`sentinel monitor dc-redis 127.0.0.1 6379 1`最后面的1就是投票的个数，也就是出现宕机的那个主库的从库，谁先得到1票，谁就当主库



## 哨兵模式搭建

使用哨兵模式的话，需要修改配置文件，因为需要设置监控哪个IP，哪个端口，多少票

哨兵的配置文件为`sentinel.conf`，但是里面很多的东西，都不用，可以重新常见一个新的文件，然后将`sentinel monitor dc-redis 哨兵监控IP 端口 投票数`复制到创建的这个配置文件当中去



1. 需要使用哨兵模式还是需要搭建一个集群，集群的搭建跟一主二从搭建一样：一台服务器模拟三台主机、查询主从信息、写操作6379、设置主从关系、全量复制、增量复制、主写从读、读写分离。

2. 创建redis_sentinel.conf文件，并编辑里边的内容：`sentinel monitor dc-redis 127.0.0.1 6379 1`，表示：指定监控主机的ip地址，port端口，得到哨兵的投票数(当哨兵投票数大于或者等于此数时切换主从关系)。

3. 新开窗口，启动哨兵

  进入`redis-sentinel`程序所在的目录，在Redis的安装目录中，启动`redis-sentinel redis_sentinel.conf`，如果不使用配置文件进行启动的话，默认是使用自己的配置文件进行启动

  ![](https://picture.xcye.xyz/wps1-1615336672415.jpg?x-oss-process=style/pictureProcess1)

  启动之后，哨兵可以查看其监控的主库的信息，包括其从库的信息(包括他们的一举一动)

  ```
  root@iZ2vc8cx7xr1j8en865pbiZ:/redis# redis-sentinel redis_sentinel.conf
  231168:X 10 Mar 2021 12:02:23.406 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
  231168:X 10 Mar 2021 12:02:23.406 # Redis version=6.2.1, bits=64, commit=00000000, modified=0, pid=231168, just started
  231168:X 10 Mar 2021 12:02:23.406 # Configuration loaded
  231168:X 10 Mar 2021 12:02:23.407 * monotonic clock: POSIX clock_gettime         监控的主机id，随机生成                        
  231168:X 10 Mar 2021 12:02:23.411 # Sentinel ID is f818191af12ce97e9d82bfaa58e6c4421f9d4fbb
  监控的主机信息
  231168:X 10 Mar 2021 12:02:23.411 # +monitor master dc-redis 127.0.0.1 6379 quorum 1
  从库信息1
  231168:X 10 Mar 2021 12:02:23.411 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ dc-redis 127.0.0.1 6379
  从库信息2
  231168:X 10 Mar 2021 12:02:23.414 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ dc-redis 127.0.0.1 6379
  ```

   

4. 主机宕机：

  ![](https://picture.xcye.xyz/wps2-1615336672415.jpg?x-oss-process=style/pictureProcess1) 

5. 等待从机投票，在sentinel窗口中查看打印信息。

  打印的信息会有几秒的延迟

  ![](https://picture.xcye.xyz/wps3-1615336672415.jpg?x-oss-process=style/pictureProcess1)

  ```
  231168:X 10 Mar 2021 12:09:01.679 # +sdown master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.679 # +odown master dc-redis 127.0.0.1 6379 #quorum 1/1
  231168:X 10 Mar 2021 12:09:01.679 # +new-epoch 1
  231168:X 10 Mar 2021 12:09:01.679 # +try-failover master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.682 # +vote-for-leader f818191af12ce97e9d82bfaa58e6c4421f9d4fbb 1
  231168:X 10 Mar 2021 12:09:01.682 # +elected-leader master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.682 # +failover-state-select-slave master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.782 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.782 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:01.837 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:02.823 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:02.823 # +failover-state-reconf-slaves master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:02.875 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:03.842 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:03.842 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:03.894 # +failover-end master dc-redis 127.0.0.1 6379
  231168:X 10 Mar 2021 12:09:03.894 # +switch-master dc-redis 127.0.0.1 6379 127.0.0.1 6380
  231168:X 10 Mar 2021 12:09:03.894 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ dc-redis 127.0.0.1 6380
  231168:X 10 Mar 2021 12:09:03.894 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ dc-redis 127.0.0.1 6380
  231168:X 10 Mar 2021 12:09:33.937 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ dc-redis 127.0.0.1 6380
  ```

   

6. 查看6380和6381的redis信息：

  ![](https://picture.xcye.xyz/wps4-1615336672415.jpg?x-oss-process=style/pictureProcess1) 

  ![img](C:\Users\chuchen\Pictures\视频截图\redis\wps5-1615336672415.jpg) 

7. 原主机恢复，启动6379：

  ![](https://picture.xcye.xyz/wps6-1615336672415.jpg?x-oss-process=style/pictureProcess1) 

  ![img](C:\Users\chuchen\Pictures\视频截图\redis\wps7-1615336672416.jpg)

  原主机如果恢复的话，那么他就会自动的变成丛机，无需用户进行设置，哨兵就可以自动设置

  ```
  Last login: Wed Mar 10 11:58:44 2021 from 39.128.212.226
  root@iZ2vc8cx7xr1j8en865pbiZ:~# ps -ef | grep redis
  root      230757  230008  0 11:57 pts/4    00:00:06 redis-server 0.0.0.0:6380
  root      230771  230008  0 11:57 pts/4    00:00:00 redis-cli -p 6380
  root      230788  230042  0 11:57 pts/5    00:00:06 redis-server 0.0.0.0:6381
  root      230805  230042  0 11:57 pts/5    00:00:00 redis-cli -p 6381
  root      230945  230907  0 11:59 pts/0    00:00:00 redis-cli -p 6381
  root      231168  230068  0 12:02 pts/6    00:00:09 redis-sentinel *:26379 [sentinel]
  root      232843  232812  0 12:28 pts/2    00:00:00 grep --color=auto redis
  ```

  通过命令`ps -ef | grep redis`可以查看信息

### 哨兵模式搭建(配置文件模式)

1—7步跟1.17.2.2一主二从搭建一样：一台服务器模拟三台主机、查询主从信息、写操作6379、设置主从关系、全量复制、增量复制、主写从读、读写分离。

8、复制三份redis_ sentinel.conf文件为redis_sentinel26379.conf、redis_sentinel26380.conf、redis_sentinel 26381.conf，并修改内容：

端口分别修改为26379、26380、26381

哨兵监控策略都修改为：

sentinel monitor mymaster 192.168.235.128 6379 2，表示：指定监控主机的ip地址，port端口，得票数多于2时表示需要切换主从关系。

如果设置密码了，都还需要设置密码：

sentinel auth-pass mymaster 123456

9、新开三个窗口，启动哨兵：./redis-sentinel ../myconfs/sentinel26379.conf

![](https://picture.xcye.xyz/wps8-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

10、主机宕机：

![img](C:\Users\chuchen\Pictures\视频截图\redis\wps9-1615336672416.jpg) 

11、等待从机投票，在sentinel窗口中查看打印信息。

![](https://picture.xcye.xyz/wps10-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

12、查看6380和6381的redis信息：

![](https://picture.xcye.xyz/wps11-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

![](https://picture.xcye.xyz/wps12-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

13、原主机恢复，：

![](https://picture.xcye.xyz/wps13-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

![](https://picture.xcye.xyz/wps14-1615336672416.jpg?x-oss-process=style/pictureProcess1) 

 

#### 小结

##### 操作：

1 查看主从复制关系命令：info replication
2 设置主从关系命令：slaveof 主机ip 主机port
3 开启哨兵模式命令：./redis-sentinel sentinel.conf
4 主从复制原则：开始是全量复制，之后是增量复制
5 哨兵模式三大任务：监控，提醒，自动故障迁移

##### 缺点 

Redis的主从复制最大的缺点就是延迟，主机负责写，从机负责备份，这个过程有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，从机器数量的增加也会使这个问题更加严重。
