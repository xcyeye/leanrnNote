# Redis 事务

Redis同样也拥有事务，只要是数据库，那么他们就拥有事务，但是Redis的事务，又和mysql等那些的事务有区别，因为Redis追求的是速度，就会失去事务的一些特性





# 事务

Redis的事务，不能像mysql那样，几个库表一起进行操作，同时成功，同时失败

Redis的事务就是，同时操作几个表，因为使用的是`select index`，那么就会将这些命令进行序列化(`排序`)，不要理解成对象的序列化，会将这些命令做一个排序，然后在根据他们的顺序的命令进行执行，如果某个命令出错，会出现两种情况

> 情况一：如果这个命令的出错，不是很严重，那么就会跳过这条出错的命令，继续执行下面的那些命令
>
> 情况二：如果这个出错的命令很严重，那么就会全部结束

这两种情况不像mysql中的那样，要么同时成功，要么同时失败，这个也是其失去的特性



## 开启事务



- #### `multi`使用此命令开启一个事务

当使用此命令开启一个事务的时候，我们就可以输入命令，但是这些输入的命令，不会立即执行，这些放入的命令会放入一个队列`queue`中，只有使用执行事务命令`exec`时，这些放在队列中的命令才会执行，但是如果放入的命令是错误命令`seta k1 v1`，那么就会报错



![](https://picture.xcye.xyz/image-20210308211224365.png?x-oss-process=style/pictureProcess1)



```
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set k1 v1
QUEUED
127.0.0.1:6379(TX)> set k2 v2
QUEUED
127.0.0.1:6379(TX)> keys *
QUEUED
127.0.0.1:6379(TX)> get k1
QUEUED
127.0.0.1:6379(TX)> set k3 v3
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) OK
3) 1) "k2"
   2) "k1"
4) "v1"
5) OK
```

这个就是开启一个事务，



## 原子性

如果执行事务的时候，出错了，那么他们的原子性是什么？

1. `放入队列的命令出错(seta k1 v1)`

    那么这个事务就不会执行，并且会操作，这个时候，就是原子性，事务中的命令同时执行成功或者同时执行失败

    ![](https://picture.xcye.xyz/image-20210308211645806.png?x-oss-process=style/pictureProcess1)

    第二条命令的时候，就出错了，所以这个事务不会执行成功，数据还是一样的

    ```
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379(TX)> set k4 v4
    QUEUED
    127.0.0.1:6379(TX)> seta k5 v5
    (error) ERR unknown command `seta`, with args beginning with: `k5`, `v5`, 
    127.0.0.1:6379(TX)> set k6 v6
    QUEUED
    127.0.0.1:6379(TX)> exec
    (error) EXECABORT Transaction discarded because of previous errors.
    127.0.0.1:6379> keys *
    1) "k3"
    2) "k2"
    3) "k1"
    ```

2. 因为事务中的命令，刚放进去的时候，是不会执行，但是会检查命令的正确性，这个就像是运行时异常和编译时异常的区别是一样的，但是如果运行事务时，命令出错`set k1 v1 incr k1`，k1是一个字符串，所以在执行的时候，必然会出错，那么对于这种情况，运行时出错，就只是出错的这个命令不会执行成功，这个事务中，其他运行正确的命令还是会执行

    ![](https://picture.xcye.xyz/image-20210308212249879.png?x-oss-process=style/pictureProcess1)

    ```
    127.0.0.1:6379> multi
    OK
    127.0.0.1:6379(TX)> set k7 v7
    QUEUED
    127.0.0.1:6379(TX)> incr k7
    QUEUED
    127.0.0.1:6379(TX)> set k8 v8
    QUEUED
    127.0.0.1:6379(TX)> keys 8
    QUEUED
    127.0.0.1:6379(TX)> exec
    1) OK
    2) (error) ERR value is not an integer or out of range
    3) OK
    4) (empty array)
    ```

    

可以通过运行结果看到，只有出错的这一条命令没有成功，其他还是正常执行



## 放弃当前事务

如果我们正在执行一个事务，但是执行到一半的时候，我们又不想执行这个事务了，那么就可以结束这个事务，并不会执行里面的命令

- `discard`使用这个命令就可以结束事务

```
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set 10 v10
QUEUED
127.0.0.1:6379(TX)> get 10
QUEUED
127.0.0.1:6379(TX)> discard
OK
127.0.0.1:6379> keys *
1) "k3"
2) "k7"
3) "k8"
4) "k1"
5) "k2"
```

使用`discard`命令之后，就退出这个事务，那么对于之后的命令，输入就会直接执行



## 监控某个字段(类似于乐观锁)



对于银行取钱，买东西这种需求，如果两个用户同时操作同一张卡，那么就难免会出现银行吃亏的情况，这种情况下，在mysql中，是使用乐观锁进行解决的，在Redis中同样也可以解决

如果监控的某个键的值发生了改变，那么这个事务就不会执行



步骤

1. 对于监控某个键，如果在执行事务的是时候，另外的用户对这个键的值进行了更改，那么监控这个字段，并且正在执行的这个事务，将不会执行

    使用`语法：watch key [key …]`进行监控某个键，这样这个键就处于被监控的状态

    ![](https://picture.xcye.xyz/image-20210308221252470.png?x-oss-process=style/pictureProcess1)

    ![](https://picture.xcye.xyz/image-20210308221429606.png?x-oss-process=style/pictureProcess1)

    因为version键的值，已经发生了改变，所以这个事务不会执行成功，输入命令`exec`返回`nil`，这是没有执行成功的标志，里面的数据并没有发生改变

2. `unwatch`

    当我们设置监控某个键之后，可以通过`unwatch`取消监控,`注意，语法不是这种 unwatch key`，只需要`unwatch`就行，是取消所有的被监控的键

但是一般不会再Redis中使用这种方式，一般都是在关系型数据库中进行操作，乐观锁



>  `语法：unwatch`

功能：清除所有先前为一个事务监控的键。

​	 如果在watch命令之后你调用了EXEC或DISCARD命令，那么就不需要手动调用UNWATCH命令。

返回值：清除成功，返回OK。

![](https://picture.xcye.xyz/wps1-1615213175436.jpg?x-oss-process=style/pictureProcess1) 





# 事务小结

1、单独的隔离操作：事务中的所有命令都会序列化、顺序地执行。事务在执行过程中，不会被其它客户端发来的命令请求所打断，除非使用watch命令监控某些键。

2、不保证事务的原子性：redis同一个事务中如果一条命令执行失败，其后的命令仍然可能会被执行，redis的事务没有回滚。Redis已经在系统内部进行功能简化，这样可以确保更快的运行速度，因为Redis不需要事务回滚的能力。





# 消息的订阅与发布

如果有几个程序员正在同时连接到这个服务器，那么如果其中一个对某个键进行了更改，其他人除了通过`get`获取值，查看时候发生了更改外，是完全不知道是否发生了更改

所以订阅就是解决这个问题

Redis消息的发布和订阅都是客户端之间，和服务器没有关系，相当于每一个客户端订阅一个频道，像音乐之声等(假如)，那么其他客户端也订阅了这个频道，那么另一个订阅这个频道的客户端发送一个消息(发送一个报文)，那么所有订阅这个频道的客户端都会收到这个消息



## Redis发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。Redis 客户端可以订阅任意数量的频道。



## Redis发布订阅示意图

图一：消息订阅者(client2 、 client5 和 client1)订阅频道 channel1：

![](https://picture.xcye.xyz/wps2-1615213783684.jpg?x-oss-process=style/pictureProcess1) 

图二：消息发布者发布消息到频道channel1，会被发送到三个订阅者：

![](https://picture.xcye.xyz/wps3-1615213783684.jpg?x-oss-process=style/pictureProcess1) 



## Redis发布订阅的常用命令

### subscribe

> 语法：`subscribe channel [channel…]`

功能：订阅一个或多个频道的信息

返回值：订阅的消息

![](https://picture.xcye.xyz/wps4-1615213783684.jpg?x-oss-process=style/pictureProcess1) 



### publish 

>  语法：`publish 频道名 message`

功能：将信息发送到指定的频道。

返回值：数字。接收到消息订阅者的数量。

![](https://picture.xcye.xyz/wps5-1615213783684.jpg?x-oss-process=style/pictureProcess1) 



只要一个客户端连接成功，使用`publish 频道名 message`就可以将这个信息发送到已经订阅这个频道的那个客户端上，那个客户端就可以收到这个消息



- 订阅频道的客户端

    ```
    127.0.0.1:6379> subscribe chu1 chu2 chu3
    Reading messages... (press Ctrl-C to quit)
    1) "subscribe"
    2) "chu1"
    3) (integer) 1
    1) "subscribe"
    2) "chu2"
    3) (integer) 2
    1) "subscribe"
    2) "chu3"
    3) (integer) 3
    1) "message"
    2) "chu1"
    3) "hello"
    1) "message"
    2) "chu2"
    3) "\xe4\xbd\xa0\xe6\x98\xaf\xe5\x88\x9d\xe4\xba\x8c\xe5\x93\x88\xe5\x93\x88\xe5\x93\x88\xe5\x93\x88"
    ```

    

- 发送信息的客户端

    ```
    127.0.0.1:6379> publish chu1 hello
    (integer) 1
    127.0.0.1:6379> publish chu2 你是初二哈哈哈哈
    (integer) 1
    127.0.0.1:6379> 
    ```

    `不能这样想，需要两个客户端都subscribe同一个频道之后，才能进行发送信息`

### psubscribe 

语法：psubscribe pattern [pattern]

功能：订阅一个或多个符合给定模式的频道。模式以 * 作为通配符，例如：news.* 匹配所有以 news. 开头的频道。

返回值：订阅的信息。

![](https://picture.xcye.xyz/wps6-1615213783685.jpg?x-oss-process=style/pictureProcess1) 
