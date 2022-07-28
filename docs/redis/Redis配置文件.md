# Redis 配置文件

## 使用配置文件启动

Redis的安装根目录下(/opt/redis-5.0.2)，Redis在启动时会加载这个配置文件，在运行时按照配置进行工作。 这个文件有时候我们会拿出来，单独存放在某一个位置，启动的时候必须明确指定使用哪个配置文件，此文件才会生效。



## 更改端口进行启动

`port 6380`

启动的时候，必须使用配置文件，否则服务器还是以默认方式进行启动



>  启动服务器

`redis-server redis.conf &`

>  `使用配置文件启动，必须要进入redis目录中，才可以，否则会报cant open file`

```
root@iZ2vc8cx7xr1j8en865pbiZ:/# redis-server redis.conf &
root@iZ2vc8cx7xr1j8en865pbiZ:/# 262885:C 07 Mar 2021 13:50:51.619 # Fatal error, can't open config file '/redis.conf': No such file or directory
```

- 进入Redis目录，输入命令启动成功，会出现Redis的标识





> 启动客户端

- 如果更改端口之后，启动客户端，不需要在进入Redis目录中，也可以

- 命令`redis-cli -p 6380`  **注意 -p是小写**

    必须要加上端口，因为不是默认启动

    

> 关闭服务器

- 命令：

    如果是使用默认方式启动，那么关闭服务器，释放端口就使用`redis-cli shutdown`

    如果是使用配置文件启动，关闭服务器，必须要加上端口`redis-cli -p 6380 shutdown`

    使用这个命令进行关闭也是需要请求，只是这种请求对于Redis来说，必须执行

    ```
    root@iZ2vc8cx7xr1j8en865pbiZ:/redis# redis-cli shutdown
    Could not connect to Redis at 127.0.0.1:6379: Connection refused
    root@iZ2vc8cx7xr1j8en865pbiZ:/redis# redis-cli -p 6380  shutdown
    1874:M 07 Mar 2021 14:53:01.023 # User requested shutdown...
    1874:M 07 Mar 2021 14:53:01.023 * Saving the final RDB snapshot before exiting.
    1874:M 07 Mar 2021 14:53:01.026 * DB saved on disk
    1874:M 07 Mar 2021 14:53:01.026 * Removing the pid file.
    1874:M 07 Mar 2021 14:53:01.026 # Redis is now ready to exit, bye bye...
    [1]+  Done                    redis-server redis.conf
    ```



如果是使用默认的端口进行关闭，那么就会使用默认的端口，和服务器都正处于监听的6379端口进行连接，所以就发送不去指令，就会报错

```
root@iZ2vc8cx7xr1j8en865pbiZ:/redis#   redis-cli shutdown
Could not connect to Redis at 127.0.0.1:6379: Connection refused 
```



`操作和关闭都需要先连接上Redis才可以`



## 更改配置文件bind后的问题



更改bind我自己IP后，可能会出现

```
 Could not create server TCP listening socket 47.108.217.74:6380: bind: Cannot assign requested address
```

出现这种问题的原因在于，客户端一直和服务器端进行连接，导致客户端(服务器)中的很多tcp没有释放，就导致端口分配不足，出现的问题





# TCP连接问题:tcp-keepalive

Redis使用的是TCP/IP进行的，每一个Redis的TCP连接数是有限的，并不是只要有一个客户端，只要知道端口，IP就一定能连接上Redis服务器，一般Redis是存在于公司，客户端是程序员个人电脑或者是程序，所以，就不一定每一个客户端，和程序都能和服务器建立连接



同时连接到Redis的TCP连接是有限的，对于不同的数据库，同一时间保持的连接数是有限的，所以对于收费的数据库，他们衡量价格的指标，很大程度上是由同一时间，能建立的连接数相关，这也就是数据库处理并发能力的强弱，因为并发，同一时间，对数据库的操作会很大



![](https://picture.xcye.xyz/image-20210307163438424.png?x-oss-process=style/pictureProcess1)

如果Redis服务器是开启的，客户端1和服务器已经建立连接，但是客户端1查完一条数据之后，就不用了，客户端没有主动退出(redis-cli -p )，那么他们之间的连接还在，只要还在，Redis服务器就会去维护他们之间的连接，所以这个也就造成了资源的浪费，

> 为了解决这个问题，Redis就有一个机制，只要和客户端建立连接，就会每隔多少秒(
>
> `tcp-keepalive 60`)向客户端发送一个请求，如果长时间客户端没有回应，那么就表明客户端不在使用，那么Redis服务器就会关闭他们之间的连接
>
> `redis.conf`中的`tcp-keepalive 60`就是配置这个间隔的时间，单位是秒数



tcp-keepalive：TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。如果设置为0，则不会进行保活检测。





# 守护线程

如果想要后台启动Redis服务，那么必须开始Redis的守护线程，默认是不开启的，在配置文件中进行更改

```
daemonize yes
```



## 前台启动和后台启动的区别



通过控制台的打印信息就可以发现他们之间的区别，

如果是前台启动

`redis-server`或者`redis-server redis.conf`(`前提是没有设置为守护线程`)那么启动的时候，就会在控制台上打印

![](https://picture.xcye.xyz/image-20210307171825162.png?x-oss-process=style/pictureProcess1)



这个就是前台启动，所以出现Redis的标识就是前台启动



后台启动

如果想要使用后台启动，那么需要在配置文件中，将其设置为守护线程

`daemonize no --> daemonize yes`

启动服务命令`redis-server redis.conf`，这样设置并且使用这个命令进行启动之后，才是后台启动

 那么不使用配置文件，并且不是守护线程启动时，`redis-server redis.conf &`和`redis-server redis.conf`启动的区别？

如果是`redis-server redis.conf`，那么在这个服务启动运行这个区间，不能在这个控制台上进行其他的操作，除非关闭服务或者使用`ctrl c`结束，才能在这个控制台上使用其他的命令

但是`redis-server redis.conf &`这个，启动成功，服务在运行期间，我们还可以在这个控制台使用

![](https://picture.xcye.xyz/image-20210308205216857.png?x-oss-process=style/pictureProcess1)

![](https://picture.xcye.xyz/image-20210308205322024.png?x-oss-process=style/pictureProcess1)





标识

如果是后台启动，那么不会打印Redis的标志，

```
root@iZ2vc8cx7xr1j8en865pbiZ:/redis# redis-server redis.conf
root@iZ2vc8cx7xr1j8en865pbiZ:/redis# 
```



通过查看后台命令`ps -ef | grep redis`就可以看到Redis服务已经运行

```
root        2708       1  0 17:23 ?        00:00:00 redis-server 0.0.0.0:6380
root        2715    2570  0 17:23 pts/1    00:00:00 grep --color=auto redis
```







# 日志信息

## 日志级别

 `loglevel`：日志级别，开发阶段可以设置成debug，生产阶段通常设置为notice或者warning.

- 总共的日志级别有

    > ```
    > # debug (a lot of information, useful for development/testing)
    > # verbose (many rarely useful info, but not a mess like the debug level)
    > # notice (moderately verbose, what you want in production probably)
    > # warning (only very important / critical messages are logged)
    > ```

    

`logfile`：指定日志文件名，如果不指定，Redis只进行标准输出。要保证日志文件所在的目录必须存在，文件可以不存在。还要在redis启动时指定所使用的配置文件，否则配置不起作用。

如果启动在这里写上一个文件，名字路径随便写，如果只是写一个名字，并且在当前路径下面没有这个文件，那么就会自动创建这个文件，并且会将日志信息打印到此文件中，如果启用，那么控制台不会打印信息

启动的时候，就会很安静

```
root@iZ2vc8cx7xr1j8en865pbiZ:/redis# redis-server redis.conf
^Croot@iZ2vc8cx7xr1j8en865pbiZ:/redis# vim redis.conf
```



`databases`：配置Redis数据库的个数，默认是16个。

默认是16个，可以通过更改配置进行，但是如果更改配置，那么就必须使用配置文件启动，否则数据库容量还是默认16个



> `如果是使用配置文件启动，那么和默认启动他们某个公共库中的数据时不同的`
>
> 比如默认启动，15库中设置k1为chu
>
> 那么使用配置文件启动，15库中的数据不是这个，可能没有(以前使用配置文件启动的时候没有设置过)



# 安全方面

Redis是不需要密码，就可以操作Redis，因为Redis是追求高效，所以这就导致了Redis的安全性比较差，如果想要让数据的安全性高，那么就使用mysql等

只要知道Redis安装的IP，和端口，那么就可以进行连接





> `requirepass`：配置Redis的访问密码。默认不配置密码，即访问不需要密码验证。此配置项需要在`protected-mode`=yes时起作用。使用密码登录客户端：redis-cli -h ip -p 6379 -a pwd



如果想要使用密码连接时，必须要设置`protected-mode`为yes，否则不会启作用，而且还需要使用配置文件进行启动服务器

使用密码进行连接时，无论是关闭还是客户端进行连接，都必须加上密码



其实，在不使用密码进行连接的时候，`protected-mode=no`应该将其设置no



# 持久化策略



## RDB



因为Redis为了能够高效的访问，所以都将数据写入内存中，但是这个也就导致了，如果服务器宕机，关机了，那么在内存中的数据就会存在丢失的情况，所以我们就应该将存储于内存中的数据写入磁盘中，这样尽管服务器处于宕机状态，那么写一次启动的时候，Redis会自动的将数据从磁盘中再次写入内存中



`RDB`就是解决这个问题，在间隔时间内，向磁盘写入数据，只要向磁盘写入数据的次数达到某个值时，就会自动触发持久化存储，`记住，是写入多少次，才触发，并不是向磁盘写入一次，就触发一次持久化存储`，这是Redis定义的一种策略



这种策略是Redis默认的，所以只要Redis的服务时开启的，那么这种策略就会启动



### 配置持久化策略

这种触发的持久化策略，可以通过配置文件进行更改

> `save <seconds> <changes>` 单位是秒
>
> 理解：多少秒内，Redis中的数据被改变了多少次

配置复合的快照触发条件，即Redis 在seconds秒内key改变changes次，Redis把快照内的数据保存到磁盘中一次。默认的策略是：

> save 3600 1    一小时改变了1次
> save 300 100   5分钟改变了100次
> save 60 10000  一分钟改变了一万次



如果要禁用Redis的持久化功能，则把所有的save配置都注释掉。

`stop-writes-on-bgsave-error`：当bgsave快照操作出错时停止写数据到磁盘，这样能保证内存数据和磁盘数据的一致性，但如果不在乎这种一致性，要在bgsave快照操作出错时继续写操作，这里需要配置为no。

`rdbcompression`：设置对于存储到磁盘中的快照是否进行压缩，设置为yes时，Redis会采用LZF算法进行压缩；如果不想消耗CPU进行压缩的话，可以设置为no，关闭此功能。

`rdbchecksum`：在存储快照以后，还可以让Redis使用CRC64算法来进行数据校验，但这样会消耗一定的性能，如果系统比较在意性能的提升，可以设置为no，关闭此功能。

、

### 保存文件



这里可以设置Redis触发持久化时，数据都保存在哪个文件中，文件名是默认的，后缀名为`rdb`

`dbfilename`：Redis持久化数据生成的文件名，默认是dump.rdb，也可以自己配置。

`dir：Redis`持久化数据生成文件保存的目录，默认是./即redis的启动目录，也可以自己配置。



这种文件，就像mysql的备份一样，我们可以将其发送给其他人使用，其他人将这个文件保存到电脑中，在Redis启动的时候，Redis就会自动读取这些文件中的数据到内存中

此文件默认是保存在当前目录下

这种策略是Redis默认的策略



## AOF

但是对于RDB，存在一个问题，如果我们修改数据的次数，并没有在配置文件中，也有可能一分钟改变了一条记录，但是半分钟又改变了数据，那么半分钟的那天记录，就不会持久化到磁盘中，因为他没有达到多长时间，改变数据的次数



所以AOF策略就可以改变这种情况

AOF的原理就是：Redis记录下`每一次`写入数据的记录(包括set，pop)，就是记录下，这些命令，将这些命令保存到磁盘中，等下次启动的时候，Redis就会运行这些命令，这个也就达到了数据的持久化，但是不会记录下`get`，因为获取数据的操作，不会对数据进行更改，AOF策略就是记录那些对数据进行改变的记录



- 修改配置文件

    ```
    appendonly no  # yes表示开启
    # The name of the append only file (default: "appendonly.aof")
    appendfilename "appendonly.aof"
    ```

    

AOF(Append Only File)，Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个***写操作***，并***追加***到文件中。Redis 重启会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

### AOF原理

Redis以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，

只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

### AOF保存的文件

AOF保存的文件是appendonly.aof文件 ,位置保存在Redis的启动目录。如果开启了AOF，Redis每次记录写操作都会往appendonly.aof文件追加新的日志内容。

### 配置AOF持久化策略

在redis.conf的“APPEND ONLY MODE”配置模块中，配置AOF保存策略。

### AOF数据恢复

通过脚本将Redis产生的appendonly.aof文件备份(cp appendonly.aof appendonly_bak.aof)，每次启动Redis前，把备份的appendonly.aof文件替换到Redis相应的目录(在redis.conf中配的的dir目录)下，只要开启AOF的功能，Redis每次启动，会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

但在实际开发中，可能因为某些原因导致appendonly.aof 文件格式异常，从而导致数据还原失败，可以通过命令redis-check-aof --fix appendonly.aof 进行修复 。会把出现异常的部分往后所有写操作日志去掉。

### AOF的重写 

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。

​	AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似。

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发。当然，也可以在配置文件中进行配置。


Redis 需要手动开启AOF持久化方式，AOF 的数据完整性比RDB高，但记录内容多了，会影响数据恢复的效率。

 

关于Redis持久化的使用：建议RDB和AOF都开启。其实RDB更适合做数据的备份，留一后手。AOF出问题了，还有RDB。

AOF与RDB模式可以同时启用，这并不冲突。如果AOF是可用的，那Redis启动时将自动加载AOF，这个文件能够提供更好的持久性保障。



`但是一般都会和关系型数据库一起使用，关系型数据库速度慢，但是可以让其保存数据，然后再将数据从关系型数据库中写入到Redis中，因为Redis默认开启的是RDB策略，所以从关系型数据库中将数据传到Redis中，用户对数据进行操作时，如果客户端达到了RDB的持久化存储的条件，那么就会将数据保存，Redis和关系型数据库之间的数据时时刻保持同步的，所以只要用户更改了一条，触发持久化存储，那么关系型数据库中也会对相应的记录进行更改`



