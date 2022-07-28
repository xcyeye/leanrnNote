---
date: 2021/12/14 22:53
title: java中的二进制，8进制，16进制表示
---

在java中，二进制，八进制，十进制，十六进制的表示分别为

- 二进制:  0b
- 八进制:  0
- 十进制：原始
- 十六进制：0x或者0X



```java
//45
int a10 = 45;

//7
int a2 = 0b110;

//108
int a8 = 0154;

//108
int a16 = 0x6C;

System.out.println("2进制 = " + a2);
System.out.println("8进制 = " + a8);
System.out.println("10进制 = " + a10);
System.out.println("16进制 = " + a16);
```

运行结果

```java
2进制 = 6
8进制 = 108
10进制 = 45
16进制 = 108
```

