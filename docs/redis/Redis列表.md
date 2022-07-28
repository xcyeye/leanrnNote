# Redis 列表

Redis列表是简单的字符串列表，按照插入顺序排序，左边（头部）、右边（尾部）或者中间都可以添加元素。链表的操作无论是头或者尾效率都极高，但是如果对中间元素进行操作，那效率会大大降低了。

列表类型的数据操作总的思想是通过key和下标操作value，key是数据标识，下标是数据在列表中的位置，value是我们感兴趣的业务数据。




## lpush



>  语法：`lpush key value [value…]`

功能：将一个或多个值 value 插入到列表 key 的最左边（表头），各个value值依次插入到表头位置。

返回值：插入之后的列表的长度。

![](https://picture.xcye.xyz/image-20210306195549303.png?x-oss-process=style/pictureProcess1)



比如存入

`lpush list1 1 2 3 4 5`

那么其的存放位置为`5 4 3 2 1`，因为第一个放入的是1，放在最右边，然后依次往左，因为是`lpush`

![](https://picture.xcye.xyz/image-20210306200002441.png?x-oss-process=style/pictureProcess1)

放入之后，还可以通过`lpush`进行放入，同样也是在之前放入的位置上，往左进行存入

```
127.0.0.1:6379[1]> lpush list1 6 7
(integer) 7
127.0.0.1:6379[1]> lrange list1 0 1
1) "7"
2) "6"
127.0.0.1:6379[1]> 
```





## `lrange`



获取指定列表中指定下标区间的元素：`lrange key startIndex endIndex`

```
127.0.0.1:6379[1]> lpush list1 1 2 3 4 5
(integer) 5
127.0.0.1:6379[1]> keys *
1) "list1"
存入的数组为5 4 3 2 1 
127.0.0.1:6379[1]> lrange list1 1 2
1) "4"
2) "3"
127.0.0.1:6379[1]> 
```

 `最左边的位置的下标为0，最右边的位置下标为-1`

```
lrange list1 0 -1
获取全部的数据
```

没有`rrange`语法，`lrange`中的l是list的缩写



## `rpush`



从往右存入，

```
127.0.0.1:6379[1]> rpush list2 1 2 3 4 5 6
下标
1 2 3 4 5 6 
0 1 2 3 4 5
(integer) 6
127.0.0.1:6379[1]> rrange list2 0 -1
(error) ERR unknown command `rrange`, with args beginning with: `list2`, `0`, `-1`, 
127.0.0.1:6379[1]> lrange list2 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"

```

`查看执行列表中的数据，不可以用rrange ，没有这种命令，都是使用lrange`进行



使用lpush之后，还可以使用rpush进行存入，他们的规则没有发生改变



## `rpop and lpop`



rpop从右边移除元素，

>  语法: `rpop key count`
>
> 如果没有写count，那么默认是移除一个，其实就相当于弹栈操作



lpop从右边移除元素，

>  语法: `lpop key count`
>
> 如果没有写count，那么默认是移除一个，其实就相当于弹栈操作

```
127.0.0.1:6379[1]> lrange list2 0 -1
1) "-3"
2) "-2"
3) "-1"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "6"
127.0.0.1:6379[1]> lpop list2 
"-3"
127.0.0.1:6379[1]> lpop list2 
"-2"
127.0.0.1:6379[1]> lrange list2 0 -1
1) "-1"
2) "1"
3) "2"
4) "3"
5) "4"
6) "5"
7) "6"
127.0.0.1:6379[1]> lpop list2  2
1) "-1"
2) "1"
127.0.0.1:6379[1]> lrange list2 0 -1
1) "2"
2) "3"
3) "4"
4) "5"
5) "6"
127.0.0.1:6379[1]> rpop list2 2
1) "6"
2) "5"
127.0.0.1:6379[1]> lrange list2 0 -1
1) "2"
2) "3"
3) "4"
```



## `lindex key index`



获取指定下标中的元素，从0开始

```
127.0.0.1:6379[1]> lrange list1 0 -1
 1) "7"
 2) "6"
 3) "5"
 4) "4"
 5) "3"
 6) "2"
 7) "1"
 8) "0"
 9) "-1"
10) "-2"
127.0.0.1:6379[1]> lindex list1 2
"5"
127.0.0.1:6379[1]> lindex list1 3
"4"
127.0.0.1:6379[1]> lindex list1 -1
"-2"
127.0.0.1:6379[1]> lindex list1 -6
"3"
127.0.0.1:6379[1]> 
```



## `lerm `

> 语法:`lrem key count value`

根据count值移除指定列表中跟value相等的数据：lrem key count value

- 如果count>0:
    - 从左边开始查找，移除count个value
- 如果count<0:
    - 从右边边开始查找，移除count个value
- 如果count=0:
    - `移除所有的value`

如果count大于其value的个数，那么会移除所有的



```
127.0.0.1:6379[1]> rpush list4 e d g h d e s s a a g d e
(integer) 13
127.0.0.1:6379[1]> lrange list4 0 -1
 1) "e"
 2) "d"
 3) "g"
 4) "h"
 5) "d"
 6) "e"
 7) "s"
 8) "s"
 9) "a"
10) "a"
11) "g"
12) "d"
13) "e"
127.0.0.1:6379[1]> lrem list4 3 e
(integer) 3
127.0.0.1:6379[1]> lrange list4 0 -1
 1) "d"
 2) "g"
 3) "h"
 4) "d"
 5) "s"
 6) "s"
 7) "a"
 8) "a"
 9) "g"
10) "d"
127.0.0.1:6379[1]> lrem list4 -1 d
(integer) 1
127.0.0.1:6379[1]> lrange list4 0 -1
1) "d"
2) "g"
3) "h"
4) "d"
5) "s"
6) "s"
7) "a"
8) "a"
9) "g"
127.0.0.1:6379[1]> lrem list4 -23 d
(integer) 2
127.0.0.1:6379[1]> lrem list4 0 s
(integer) 2
127.0.0.1:6379[1]> lrange list4 0 -1
1) "g"
2) "h"
3) "a"
4) "a"
5) "g"
127.0.0.1:6379[1]> 
```



## llen

语法：llen key

功能：获取列表 key 的长度

返回值：数值，列表的长度；key不存在返回0

![](https://picture.xcye.xyz/wps18.jpg?x-oss-process=style/pictureProcess1) 



## ltrim

语法：ltrim key startIndex endIndex

功能：截取key的指定下标区间的元素，并且赋值给key。下标从0开始，一直到列表长度-1；下标也可以是负数，表示列表从后往前取，-1表示倒数第一个元素，-2表示倒数第二个元素，以此类推；startIndex和endIndex超出范围不会报错。

返回值：执行成功返回ok

![](https://picture.xcye.xyz/wps20.jpg?x-oss-process=style/pictureProcess1) 



## lset

语法：lset key index value

功能：将列表 key 下标为 index 的元素的值设置为 value。

功能：设置成功返回ok ; key不存在或者index超出范围返回错误信息。

![](https://picture.xcye.xyz/wps21.jpg?x-oss-process=style/pictureProcess1) 



## linsert

语法：linsert key before/after pivot value

功能：将值 value 插入到列表 key 当中位于值 pivot 之前或之后的位置。key不存在或者pivot不在列表中，不执行任何操作。

返回值：命令执行成功，返回新列表的长度。没有找到pivot返回 -1， key不存在返回0。

![](https://picture.xcye.xyz/wps22.jpg?x-oss-process=style/pictureProcess1) 
