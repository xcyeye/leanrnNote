# 递归

## 递归JVM初识

![](https://picture.xcye.xyz/image-20211018203057464.png?x-oss-process=style/pictureProcess1)

上面程序就是一个简单的递归操作

其在内存中的执行过程如下图所示

![](https://picture.xcye.xyz/image-20211018203235368.png?x-oss-process=style/pictureProcess1)



::: tip 

1. 首先main方法会先入栈，当调佣test方法的时候，test方法也会进行入栈
2. 每一次递归调用，都会进行入栈操作

需要理清楚递归的调用顺序

`一定要给递归设置一个条件，如果递归一直执行下去的话，那么此程序就会出现栈溢出的情况`

```java
Exception in thread "main" java.lang.StackOverflowError
	at vin.cco.recursion.RecursionTest1.test(RecursionTest1.java:15)
	at vin.cco.recursion.RecursionTest1.test(RecursionTest1.java:15)

```

:::
