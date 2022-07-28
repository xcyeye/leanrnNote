---
date: 2021/12/14 20:28
---

berak语句，只会终止当前循环，如果是嵌套循环的话，执行break之后，会跳到外部的循环中，并不是直接退出所有循环

```java
for (int i = 0; i < 3; i++) {
    for (int a = 0; a < 5; a++) {
        if (a == 3) {
            break;
        }
        System.out.println(a + " --> a");
    }
    System.out.println(i + " ---> i");
}
```

> 当执行的时候，a值等于3的时候，那么就会退出`for (int a = 0; a < 5; a++) {`这个循环，但是并不会连着`for (int i = 0; i < 3; i++) {`也退出
>
> `break如果没带标签的时候，只是退出当前循环，而不是所有的循环`



那么我们就可以使用待标签的break，这样我们就可以退出指定循环了

```java
first:for (int i = 0; i < 3; i++) {
    for (int a = 0; a < 5; a++) {
        if (a == 3) {
            break first;
        }
        System.out.println(a + " --> a");
    }
    System.out.println(i + " ---> i");
}
```

