# Redis5种数据结构



程序是用来处理数据的，Redis是用来存储数据的；程序处理完的数据要存储到redis中，不同特点的数据要存储在Redis中不同类型的数据结构中。

## 字符串类型 string

字符串类型是Redis中最基本的数据结构，它能存储任何类型的数据，包括二进制数

据，序列化后的数据，JSON化的对象甚至是一张图片。最大512M。

`这种存储方式，一个K只能存储一个value`

![](https://picture.xcye.xyz/wps7.jpg?x-oss-process=style/pictureProcess1) 

## 列表类型  list

Redis列表是简单的字符串列表，按照插入顺序排序，元素可以重复。你可以添加一个元素到列表的头部（左边）或者尾部（右边）,底层是个链表结构。

`一个K可以存储多个value`，并且是有序的，因为取的时候是通过下标进行取的

这是一种链表的结构

像java中的那样，通过K找到存储在哪个下标，然后在添加到这个下标上的链表上

![](https://picture.xcye.xyz/wps8.jpg?x-oss-process=style/pictureProcess1) 

## 集合类型 set

Redis的Set是string类型的无序无重复集合。

`一个K，多个value`，这是一种无序的，但是存储的数据是有序，存进去的数据顺序和取出来的循序是不同的

![](https://picture.xcye.xyz/wps9.jpg?x-oss-process=style/pictureProcess1) 

## 哈希类型  hash

​	Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

![](https://picture.xcye.xyz/wps10.jpg?x-oss-process=style/pictureProcess1) 

## 有序集合类型 zset (sorted set)

Redis 有序集合zset和集合set一样也是string类型元素的集合，且不允许重复的成员。

不同的是zset的每个元素都会关联一个分数（分数可以重复），redis通过分数来为集合中的成员进行从小到大的排序。

`无序，存进去的顺序和取出来的顺序不用，但是里面的数据，是按照一定的顺序进行存储的`

![](https://picture.xcye.xyz/wps11.jpg?x-oss-process=style/pictureProcess1) 



> 字符串:zhangsan  20   true                        tring 单key:单value: username:zhangsan age:20
> list列表:13900009999 zs@163.com 321321 list   单key:多有序value: contacts:13900009999,xxx,xxxx
> set集合:beijing shanghai chongqing tianjin set    单key:多无序value:city:bj sh cq tj
> pojo:id :1001,name:zhangsan,age:20          hash   单key: 对象(属性:值):
>                                                                student:id:1001,name:zhangsan,age:20
>    zset   单key:多有序vlaue:
> city: 
> 1000 tj,1200 cq,1500 sh,2000 bj





# Redis中key的常用命令

查询Redis库中的key值，可以使用命令

```
语法：keys pattern
作用：查找所有符合模式pattern的key.  pattern可以使用通配符。
通配符：
*：表示0或多个字符，例如：keys * 查询所有的key。
？：表示单个字符，例如：wo?d , 匹配 word , wood
[] ：表示选择[]内的一个字符，例如wo[or]d, 匹配word, wood, 不匹配wold、woord
```





## 查询所有的key(`*号可以是一个，也可以查看多个`)

    所有：`keys *`
    
    单独一个：`keys k`
    
    ```
    127.0.0.1:6379> keys c*
    1) "chuchen"
    2) "chun"
    查询以c开头的所有的key
    
    127.0.0.1:6379> keys c*n
    1) "chuchen"
    2) "chun"
    查询以c开头，第二个字母随意，以n结尾的key
    ```

## ?号通配符

    ```
    127.0.0.1:6379> keys ch?n
    1) "chun"
    ```
    
    `?`号就像mysql中的`_`只能匹配一个字符

## `[]`通配符

    这个就像mysql中的`in`，从`[]`中匹配一个进行查找
    
    ```
    127.0.0.1:6379> keys he[lfd]lo
    1) "hello"
    从[lfd]每一个进行查找匹配
    ```

## `exists`判断某个key值是否存在

    > 存在：返回1
    >
    > 单个不存在：0
    >
    > 判断多个：返回存在的个数
    
    ![image-20210306161821509](C:\Users\chuchen\Pictures\视频截图\redis\image-20210306161821509.png)

## `move key index`

    将键`key`移动到index库中

## `ttl key`

    查看key值的剩余生存时间，以秒为单位，因为key是保存在内存中，可以设置key值在内存中的时间
    
    如果key值不存在，那么返回`-2`
    
    如果返回`-1`，则代表永久存活，默认就是-1

## 设置key值的剩余时间

    >  语法：expire key seconds
    
    作用：设置key的生存时间，超过时间，key自动删除。单位是秒。
    
    返回值：设置成功返回数字 1，其他情况是 0 。

## 语法：type key

    作用：查看key所存储值的数据类型
    
    返回值：字符串表示的数据类型

   >  none (key不存在)
>
   >  string (字符串)
>
   >  list (列表)
>
   >  set (集合)
>
   >  zset (有序集)
>
   >  hash (哈希表)

## rename key newkey
    作用：将key改为名newkey。当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。
    
    当 newkey 已经存在时， RENAME 命令将覆盖旧值。

![](https://picture.xcye.xyz/wps1-1615022381305.jpg?x-oss-process=style/pictureProcess1) 

## del

语法：del key [key…]

作用：删除存在的key，不存在的key忽略。

返回值：数字，删除的key的数量。

![](https://picture.xcye.xyz/wps2-1615022381304.jpg?x-oss-process=style/pictureProcess1) 



# String类型



字符串类型是Redis中最基本的数据类型，它能存储任何形式的字符串，包括二进制数

据，序列化后的数据，JSON化的对象甚至是一张图片。

字符串类型的数据操作总的思想是通过key操作value，key是数据标识，value是我们感

兴趣的业务数据。

## set

>  语法：set key value

功能：将字符串值 value 设置到 key 中，如果key已存在，后放的值会把前放的值覆盖掉。

返回值：OK表示成功

![](https://picture.xcye.xyz/wps3-1615025468036.jpg?x-oss-process=style/pictureProcess1) 

![](https://picture.xcye.xyz/wps4-1615025468036.jpg?x-oss-process=style/pictureProcess1) 

## get

>  语法：get key

功能：获取 key 中设置的字符串值

返回值：key存在，返回key对应的value；

​		key不存在，返回nil

![](https://picture.xcye.xyz/wps5-1615025468037.jpg?x-oss-process=style/pictureProcess1) 

## append

>  语法：append key value

功能：如果 key 存在，则将 value 追加到 key 原来旧值的末尾

​	 如果 key 不存在，则将key 设置值为 value

返回值：追加字符串之后的总长度(字符个数)

![](https://picture.xcye.xyz/wps6-1615025468037.jpg?x-oss-process=style/pictureProcess1) 

## strlen

>  语法：strlen key

功能：返回 key 所储存的字符串值的长度

返回值：如果key存在，返回字符串值的长度；

​		key不存在，返回0

![](https://picture.xcye.xyz/wps7-1615025468037.jpg?x-oss-process=style/pictureProcess1) 


## incr
语法：incr key

功能：将key中储存的数字值加1，如果key不存在，则key的值先被初始化为0再执行incr操作。

返回值：返回加1后的key值

![](https://picture.xcye.xyz/wps8-1615027305590.jpg?x-oss-process=style/pictureProcess1) 

## decr

语法：decr key

功能：将 key 中储存的数字值减1，如果 key 不存在，则么 key 的值先被初始化为 0 再执行 decr 操作。

返回值：返回减1后的key值

![](https://picture.xcye.xyz/wps9-1615027305591.jpg?x-oss-process=style/pictureProcess1) 

## incrby

语法：incrby key offset

功能：将 key 所储存的值加上增量值，如果 key 不存在，则 key 的值先被初始化为 0 再执行 INCRBY 命令。

返回值：返回增量之后的key值。

![](https://picture.xcye.xyz/wps10-1615027305591.jpg?x-oss-process=style/pictureProcess1) 

## decrby

语法：decrby key offset

功能：将 key 所储存的值减去减量值，如果 key 不存在，则 key 的值先被初始化为 0 再执行 DECRBY 命令。

返回值：返回减量之后的key值。

![](https://picture.xcye.xyz/wps11-1615027305591.jpg?x-oss-process=style/pictureProcess1) 

## getrange

语法：getrange key startIndex endIndex

功能：获取 key 中字符串值从 startIndex 开始到 endIndex 结束的子字符串,包括startIndex和endIndex, 负数表示从字符串的末尾开始，-1 表示最后一个字符。

`如果endInxex`的长度大于字符串长度的时候，不会报错，会截取全部的字符串

![](https://picture.xcye.xyz/wps12.jpg?x-oss-process=style/pictureProcess1) 

## setrange

语法：setrange key offsetIndex value

功能：用value覆盖key的存储的值从offset开始。

返回值：修改后的字符串的长度。

![](https://picture.xcye.xyz/wps13.jpg?x-oss-process=style/pictureProcess1)



`如果此key不存在的话，那么会set这个key，value就是其值，并且从offsetIndex处开始，将value值填入此key中，offsetIndex前面的几位，使用“\x00”`代替
>
> ```
> 127.0.0.1:6379> setrange f2 1 cfcf
> (integer) 5
> 127.0.0.1:6379> get f2
> "\x00cfcf"
> 127.0.0.1:6379> 
> ```



## setex

语法：setex key seconds value

功能：设置key的值，并将 key 的生存时间设为 seconds (以秒为单位)  ，如果key已经存在，将覆盖旧值。

返回值：设置成功，返回OK。

![](https://picture.xcye.xyz/wps14.jpg?x-oss-process=style/pictureProcess1) 

## setnx

语法：setnx key value

功能：setnx 是 set if not exists 的简写，如果key不存在，则 set 值，存在则不设置值。

返回值：设置成功，返回1

设置失败，返回0

![](https://picture.xcye.xyz/wps15.jpg?x-oss-process=style/pictureProcess1) 

## mset

语法：mset key value [key value…]

功能：同时设置一个或多个 key-value 对

返回值：设置成功，返回OK。

![](https://picture.xcye.xyz/wps16.jpg?x-oss-process=style/pictureProcess1) 

## mget

语法：mget key [key …]

功能：获取所有(一个或多个)给定 key 的值

返回值：包含所有key的列表，如果key不存在，则返回nil。

![](https://picture.xcye.xyz/wps17.jpg?x-oss-process=style/pictureProcess1) 

## msetnx

语法：msetnx key value[key value…]

功能：同时设置一个或多个 key-value 对，如果有一个key是存在的，则设置不成功。

返回值：设置成功，返回1

设置失败，返回0
