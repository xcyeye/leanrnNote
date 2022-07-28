# Redis set集合

Redis的Set是string类型的无序不重复集合。

集合类型的数据操作总的思想是通过key确定集合，key是集合标识，元素没有下标，只有直接操作业务数据和数据的个数。



# 常用命令

## sadd 

语法：sadd key member [member…]

功能：将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略，不会再加入。

返回值：加入到集合的新元素的个数(不包括被忽略的元素)。

![](https://picture.xcye.xyz/wps23.jpg?x-oss-process=style/pictureProcess1) 

## smembers

语法：smembers key

功能：获取集合 key 中的所有成员元素，不存在的key视为空集合。

返回值：返回指定集合的所有元素集合，不存在的key，返回空集合。

![](https://picture.xcye.xyz/wps24.jpg?x-oss-process=style/pictureProcess1)



```
127.0.0.1:6379[1]> sadd set1 s d g d s e ddf s
(integer) 5
127.0.0.1:6379[1]> smembers set1
1) "s"
2) "ddf"
3) "d"
4) "g"
5) "e"
127.0.0.1:6379[1]> sadd set1 s f u 
(integer) 2
127.0.0.1:6379[1]> smembers set1
1) "ddf"
2) "u"
3) "f"
4) "s"
5) "d"
6) "g"
7) "e"
```





## sismember

语法：sismember key member

功能：判断 member 元素是否是集合 key 的元素

返回值：member是集合成员返回1，其他返回 0 。

![](https://picture.xcye.xyz/wps25.jpg?x-oss-process=style/pictureProcess1) 






## scard

语法：scard key

功能：获取集合里面的元素个数

返回值：数字，key的元素个数。其他情况返回 0 。

![](https://picture.xcye.xyz/wps26.jpg?x-oss-process=style/pictureProcess1) 






## srem

语法：srem key member [member…]

功能：移除集合中一个或多个元素，不存在的元素被忽略。

返回值：数字，成功移除的元素个数，不包括被忽略的元素。

![](https://picture.xcye.xyz/wps27.jpg?x-oss-process=style/pictureProcess1) 






## srandmembe

语法：srandmember key[count]

功能：只提供key，随机返回集合中一个元素，元素不删除，依然在集合中；

​	 提供了count时，count 正数, 返回包含count个数元素的集合，集合元素各不重复。count是负数，返回一个count绝对值的长度的集合，集合中元素可能会重复多次。

返回值：一个元素或者多个元素的集合

![](https://picture.xcye.xyz/wps28.jpg?x-oss-process=style/pictureProcess1) 





## spop

语法：spop key[count]

功能：随机从集合中删除一个或count个元素。

返回值：被删除的元素，key不存在或空集合返回nil。

![](https://picture.xcye.xyz/wps29.jpg?x-oss-process=style/pictureProcess1) 






## smove

语法：smove src dest member

功能：将 member 元素从src集合移动到dest集合，member不存在，smove不执行操作，返回0，如果dest存在member，则仅从src中删除member。

返回值：成功返回 1 ，其他返回 0 。



注意：这里并不是`srem`

![](https://picture.xcye.xyz/wps30.jpg?x-oss-process=style/pictureProcess1) 





## sdiff

语法：sdiff key key [key…]

功能：返回指定集合的差集，以第一个集合为准进行比较，即第一个集合中有但在其它任何集合中都没有的元素组成的集合。

返回值：返回第一个集合中有而后边集合中都没有的元素组成的集合，如果第一个集合中的元素在后边集合中都有则返回空集合。

![](https://picture.xcye.xyz/wps31.jpg?x-oss-process=style/pictureProcess1) 





## sinte

语法：sinter key key [key…]

功能：返回指定集合的交集，即指定的所有集合中都有的元素组成的集合。

返回值：交集元素组成的集合，如果没有则返回空集合。

![](https://picture.xcye.xyz/wps32.jpg?x-oss-process=style/pictureProcess1) 






## sunion

语法：sunion key key [key…]

功能：返回指定集合的并集，即指定的所有集合元素组成的大集合，如果元素有重复，则保留一个。

返回值：返回所有集合元素组成的大集合，如果所有key都不存在，返回空集合。

![](https://picture.xcye.xyz/wps33.jpg?x-oss-process=style/pictureProcess1) 
