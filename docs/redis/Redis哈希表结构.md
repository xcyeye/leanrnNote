# 哈希类型(hash)

Redis的hash 是一个string类型的key和value的映射表，这里的value是一系列的键值对，hash特别适合用于存储对象。

`key值可以存放对象名，对象里面有很多的属性，每一个属性都有一个值，也就是hash中的key value形式`

哈希类型的数据操作总的思想是通过key和field操作value，key是数据标识，field是域，value是我们感兴趣的业务数据。


# 基本操作命令

<br>

## hset

> 语法：`hset  key  field  value [field  value …]`

功能：将键值对field-value设置到哈希列表key中，如果key不存在，则新建哈希列表，然后执行赋值，如果key下的field已经存在，则value值覆盖。

返回值：返回设置成功的键值对个数。

![](https://picture.xcye.xyz/wps2-1615082512733.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> hset h1 id 1 name chuchen age 22 blog vipblogs.cn
(integer) 4
127.0.0.1:6379[1]> hset h2 name cc habit coding
(integer) 2
```





## hget

> 语法：`hget key field`

功能：获取哈希表 key 中给定域 field 的值。

返回值：field域的值，如果key不存在或者field不存在返回nil。

![](https://picture.xcye.xyz/wps3-1615082512733.jpg?x-oss-process=style/pictureProcess1) 





## hmset

> 语法：`hmset key  field value [field value…]`

功能：同时将多个 field-value (域-值)设置到哈希表 key 中，此命令会覆盖已经存在的field，hash表key不存在，创建空的hash表，再执行hmset.

返回值：设置成功返回ok，如果失败返回一个错误。

![](https://picture.xcye.xyz/wps4-1615082512734.jpg?x-oss-process=style/pictureProcess1) 





## hmget

> 语法：`hmget key field [field…]`

功能：获取哈希表 key 中一个或多个给定域的值

返回值：返回和field顺序对应的值，如果field不存在，返回nil。

![](https://picture.xcye.xyz/wps5-1615082512734.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> hmget h1 id age name
1) "1"
2) "22"
3) "chuchen"
127.0.0.1:6379[1]> hmget h2 habit drink 
1) "computer"
2) "water"
```





## hgetall

> 语法：`hgetall key`

功能：获取哈希表 key 中所有的域和值

返回值：以列表形式返回hash中域和域的值，key不存在，返回空hash.

![](https://picture.xcye.xyz/wps6-1615082512734.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> hgetall h1
1) "id"
2) "1"
3) "name"
4) "chuchen"
5) "age"
6) "22"
7) "blog"
8) "vipblogs.cn"
127.0.0.1:6379[1]> hgetall h2
1) "name"
2) "cc"
3) "habit"
4) "computer"
5) "drink"
```





## hdel

> 语法：`hdel key field [field…]`

功能：删除哈希表 key 中的一个或多个指定域field，不存在field直接忽略。

返回值：成功删除的field的数量。

![](https://picture.xcye.xyz/wps7-1615082512734.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> hgetall h2
1) "name"
2) "cc"
3) "habit"
4) "computer"
5) "drink"
6) "water"
127.0.0.1:6379[1]> hdel h2 name habit
(integer) 2
127.0.0.1:6379[1]> hgetall h2
1) "drink"
2) "water"
```





## hlen

> 语法：`hlen key`

功能：获取哈希表 key 中域field的个数

返回值：数值，field的个数。key不存在返回0.

![](https://picture.xcye.xyz/wps8-1615082512734.jpg?x-oss-process=style/pictureProcess1) 
```
127.0.0.1:6379[1]> hgetall h1
1) "id"
2) "1"
3) "name"
4) "chuchen"
5) "age"
6) "22"
7) "blog"
8) "vipblogs.cn"
127.0.0.1:6379[1]> hlen h1
(integer) 4
```





## hexists

> 语法：`hexists key field`

功能：查看哈希表 key 中，给定域 field 是否存在

返回值：如果field存在，返回1，其他返回0。

![](https://picture.xcye.xyz/wps9-1615082512734.jpg?x-oss-process=style/pictureProcess1) 





## hkeys

> 语法：`hkeys key`

功能：查看哈希表 key 中的所有field域列表

返回值：包含所有field的列表，key不存在返回空列表

![](https://picture.xcye.xyz/wps10-1615082512734.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> hkeys h1
1) "id"
2) "name"
3) "age"
4) "blog"
```





## hvals

> 语法：`hvals key`

功能：返回哈希表 中所有域的值列表

返回值：包含哈希表所有域值的列表，key不存在返回空列表。

![](https://picture.xcye.xyz/wps11-1615082512734.jpg?x-oss-process=style/pictureProcess1) 
```
127.0.0.1:6379[1]> hvals h1
1) "1"
2) "chuchen"
3) "22"
4) "vipblogs.cn"
```
`不包含key值`





## hincrby 

> 语法：`hincrby key field int`

功能：给哈希表key中的field域增加int

返回值：返回增加之后的field域的值

![](https://picture.xcye.xyz/wps12-1615082512734.jpg?x-oss-process=style/pictureProcess1) 
```
127.0.0.1:6379[1]> hincrby h1 age 199
(integer) 222
```





## hincrbyfloat

> 语法：`hincrbyfloat key field float`

功能：给哈希表key中的field域增加float

返回值：返回增加之后的field域的值

![](https://picture.xcye.xyz/wps13-1615082512734.jpg?x-oss-process=style/pictureProcess1) 





## hsetnx 

> 语法：`hsetnx key field value`

功能：将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在的时候才设置，否则不设置。

返回值：设值成功返回1，其他返回0.

![](https://picture.xcye.xyz/wps14-1615082512734.jpg?x-oss-process=style/pictureProcess1) 

