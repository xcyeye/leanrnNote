 # Redis zset集合


Redis 有序集合zset和集合set一样也是string类型元素的集合，且不允许重复的成员。

不同的是zset的每个元素都会关联一个分数（分数可以重复），redis通过分数来为集合中的成员进行从小到大的排序。
如果分数相同的话，那么分数相同的元素，排序就是随机进行排

有序集合也是有下标的，但是这个和列表是不同的，列表中的下标是根据存入的顺序就行，而有序集合中的下标是根据分数排序之后的位置

有序集合排序默认是从小到大的顺序


#  基本操作命令


## zadd

> 语法：`zadd key score member [score member…]`

这里的`score`就是分数，在使用zadd进行添加的时候，必须为一个member指定score值，可以重复，程序会根据score的值，进行排序

功能：将一个或多个 member 元素及其 score 值加入到有序集合 key 中，如果member存在集合中，则覆盖原来的值；score可以是整数或浮点数.

返回值：数字，新添加的元素个数.

![](https://picture.xcye.xyz/wps15-1615085341294.jpg?x-oss-process=style/pictureProcess1) 





## range

>  语法：`zrange key startIndex endIndex [WITHSCORES]`

功能：查询有序集合，指定区间的内的元素。集合成员按score值从小到大来排序；startIndex和endIndex都是从0开始表示第一个元素，1表示第二个元素，以此类推； startIndex和endIndex都可以取负数，表示从后往前取，-1表示倒数第一个元素；WITHSCORES选项让score和value一同返回。

返回值：指定区间的成员组成的集合。

![](https://picture.xcye.xyz/wps16-1615085341295.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> zadd z1 20 k1 10 k2 80 k3 1 k4 10000 k5
(integer) 5
127.0.0.1:6379[1]> zrange z1 0 -1
1) "k4"
2) "k2"
3) "k1"
4) "k3"
5) "k5"
127.0.0.1:6379[1]> zrange z1 -1 0
(empty array)
不可以使用这种方式zrange z1 -1 0
127.0.0.1:6379[1]> zrange z1 0 -1 withscores
 1) "k4"
 2) "1"
 3) "k2"
 4) "10"
 5) "k1"
 6) "20"
 7) "k3"
 8) "80"
 9) "k5"
10) "10000"
```





## zrangebyscore

> 语法：`zrangebyscore key min max [WITHSCORES ] [LIMIT offset count]`

功能：获取有序集 key 中，所有 score 值介于 min 和 max 之间（包括min和max）的成员，有序成员是按递增（从小到大）排序；

符号：
- 使用符号”(“ 表示包括min但不包括max；`(30表示大于30，不包括30在内`
```
127.0.0.1:6379[1]> zrange z1 0 -1 withscores
 1) "k4"
 2) "1"
 3) "k2"
 4) "10"
 5) "k1"
 6) "20"
 7) "k3"
 8) "80"
 9) "k5"
10) "10000"
127.0.0.1:6379[1]> zrangebyscore z1 10 80
1) "k2"
2) "k1"
3) "k3"
127.0.0.1:6379[1]> zrangebyscore z1 (20 80
1) "k3"
```

- withscores 显示score和 value；
```
127.0.0.1:6379[1]> zrangebyscore z1 (20 80 withscores
1) "k3"
2) "80"
```

- limit用来限制返回结果的数量和区间，在结果集中从第offset个开始，取count个。
相当于mysql中的分页，下标从0开始
```
127.0.0.1:6379[1]> zrangebyscore z1 (19 (90 withscores
 1) "k1"
 2) "20"
 3) "k6"
 4) "45"
 5) "k8"
 6) "54"
 7) "k3"
 8) "80"
 9) "k9"
10) "89"
127.0.0.1:6379[1]> zrangebyscore z1 (19 (90 withscores limit 2 2 
1) "k8"
2) "54"
3) "k3"
4) "80"
```

返回值：指定区间的集合数据

![](https://picture.xcye.xyz/wps17-1615085341295.jpg?x-oss-process=style/pictureProcess1) 

![](https://picture.xcye.xyz/wps18-1615085341295.jpg?x-oss-process=style/pictureProcess1) 





## zrem

> 语法：`zrem key member [member…]`

功能：删除有序集合 key 中的一个或多个成员，不存在的成员被忽略。

返回值：被成功删除的成员数量，不包括被忽略的成员。

![](https://picture.xcye.xyz/wps19-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zcard

> 语法：`zcard key`

作用：获取有序集 key 的元素成员的个数。

返回值：key存在，返回集合元素的个数； key不存在，返回0。

![](https://picture.xcye.xyz/wps20-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zcount

> 语法：`zcount key min max`

功能：返回有序集 key 中， score 值在 min 和 max 之间(包括 score 值等于 min 或 max )的成员的数量。

返回值：指定有序集合中分数在指定区间内的元素数量。

`(`符号也是可以使用的
```
127.0.0.1:6379[1]> zrange z1 0 -1 withscores
 1) "k4"
 2) "1"
 3) "k2"
 4) "10"
 5) "k1"
 6) "20"
 7) "k6"
 8) "45"
 9) "k8"
10) "54"
11) "k3"
12) "80"
13) "k9"
14) "89"
15) "k7"
16) "324"
17) "k5"
18) "10000"
127.0.0.1:6379[1]> zcount z1 20 89
(integer) 5
127.0.0.1:6379[1]> zcount z1 (89 100000
(integer) 2
127.0.0.1:6379[1]> zcount z1 89 100000
(integer) 3
```

![](https://picture.xcye.xyz/wps21-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zrank

> 语法：`zrank key member`

功能：获取有序集 key 中成员 member 的排名，有序集成员按 score 值从小到大顺序排列，从0开始排名，score最小的是0 。

返回值：指定元素在有序集合中的排名；如果指定元素不存在，返回nil。

`下标从0开始`

![](https://picture.xcye.xyz/wps22-1615085341300.jpg?x-oss-process=style/pictureProcess1) 

```
127.0.0.1:6379[1]> zrange z1 0 -1 withscores
 1) "k4"
 2) "1"
 3) "k2"
 4) "10"
 5) "k1"
 6) "20"
 7) "k6"
 8) "45"
 9) "k8"
10) "54"
11) "k3"
12) "80"
13) "k9"
14) "89"
15) "k7"
16) "324"
17) "k5"
18) "10000"
127.0.0.1:6379[1]> zrank z1 k4
(integer) 0
127.0.0.1:6379[1]> zrank z1 k2
(integer) 1
```





## zscore

> 语法：`zscore key member`

功能：获取有序集合key中元素member的分数。

返回值：返回指定有序集合元素的分数。

![](https://picture.xcye.xyz/wps23-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zrevrank

> 语法：`zrevrank key member`

功能：获取有序集 key 中成员 member 的排名，有序集成员按 score 值从大到小顺序排列，从0开始排名，score最大的是0 。

返回值：指定元素在有序集合中的排名；如果指定元素不存在，返回nil。

![](https://picture.xcye.xyz/wps24-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zrevrange 

> 语法：`zrevrange key startIndex endIndex [WITHSCORES]`

功能：查询有序集合，指定区间的内的元素。集合成员按score值从大到小来排序；startIndex和endIndex都是从0开始表示第一个元素，1表示第二个元素，以此类推； startIndex和endIndex都可以取负数，表示从后往前取，-1表示倒数第一个元素；WITHSCORES选项让score和value一同返回。

返回值：指定区间的成员组成的集合。

![](https://picture.xcye.xyz/wps25-1615085341300.jpg?x-oss-process=style/pictureProcess1) 





## zrevrangebyscore

> 语法：`zrevrangebyscore key max min  [WITHSCORES ] [LIMIT offset count]`

功能：获取有序集 key 中，所有 score 值介于 max 和 min 之间（包括max和min）的成员，有序成员是按递减（从大到小）排序；

​	 使用符号`(`表示不包括min和max；

​	 withscores 显示score和 value；

   limit用来限制返回结果的数量和区间，在结果集中从第offset个开始，取count个。

返回值：指定区间的集合数据

![](https://picture.xcye.xyz/wps26-1615085341300.jpg?x-oss-process=style/pictureProcess1) 
