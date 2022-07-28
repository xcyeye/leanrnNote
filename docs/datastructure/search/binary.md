---
date: "2021/10/23 17:09"
keyword: '二分查找,Java二分查找,Java差值查找,Java数据结构,数据结构,差值查找,Java二分查找实现'
description: "插值查找算法类似于二分查找，不同的是插值查找每次从自适应mid处开始查找。将折半查找中的求mid 索引的公式 , low 表示左边索引left, high表示右边索引right.key 就是前面我们讲的  findVal差值查找，其就是解决二分法，对于数组元素比较均匀，如1到100的元素，其可以很快的找到(`经过测试，一次就可以找打这个位置`)"
---

# 二分查找

在使用二分查找的时候，一定要保证数组是一个有序的，无论是正序还是倒叙都行，否则的话，就不能使用二分查找

## 思想

假设一个有序数组

```java
int[] arr = {1,8, 10, 16,56,60,87,89};
```

查找的值为`60`，那么其执行的过程就是

::: tip

1. 先找出该有序数组的中间值`56`
2. 使用查找值`60`和中间值`56`进行比较，如果查找值比中间值大，那么使用递归，向右进行查找，这个时候的中间值就为`87`
3. 使用查找值`60`和中间值`56`进行比较，如果查找值比中间值小，那么使用递归，向左进行查找，这个时候的中间值就为`10`
4. 以此类推，使用递归

:::



## 代码

```java
int[] arr = {1,8, 10, 16,56,60,87,89};
int binarySearch = binarySearch(arr, 0, arr.length -1, 11);

public static int binarySearch(int[] arr, int left, int right, int value) {
    int middleIndex = (left + right) / 2;
    int middle = arr[middleIndex];

    if (left > right) {
        return -1;
    }

    if (value > middle) {
        //在右边进行查找
        return binarySearch(arr, middleIndex + 1,right,value);
    }else if (value < middle) {
        return binarySearch(arr, left,middleIndex - 1,value);
    }else {
        return middleIndex;
    }
}
```

> 因为使用的是递归，所以需要设置退出的条件，可以分为两种
>
> - 查找到值，直接返回middle
> - 没有查找到值
>
> `如果我们需要查找的值，在该数组中，并不存在，那么就会使用递归，递归一遍该数组中的元素，所以这个的判断条件就是 left > right`
>
> > 如果查找值，比中间值大，那么left值，就会一直 `+ 1`，如果比中间值小，那么`right`值，就会一直` - 1`，所以当`left > right`的时候，表示已经查找完整个数组了



## 问题

使用这个二分查找，如果对于一个连续的数组，比如一个从1到100的数组，我们需要查找的值，是100，或者是1，那么使用二分查找，其需要执行几次，第一次找到55，第二次在找到中间值，然后...

::: tip

所以对于这种数据量较大，关键字(`数组内的元素`)分布比较均匀的查找表来说，采用插值查找, 速度较快.

> `关键字分布不均匀的情况下，该方法不一定比折半查找要好`

:::



## 差值查找

差值查找，其就是解决二分法，对于数组元素比较均匀，如1到100的元素，其可以很快的找到(`经过测试，一次就可以找打这个位置`)



::: tip

差值查找每次从`自适应`middleIndex处开始查找，能够很快就找到

:::



> - 重新定义中间值下标公式
>   - 二分法 `(left + right) /2`
>   - 差值查找`left + (right - left) * (value - arr[left]) / (arr[right] - arr[left])`



### 代码实现

```java
public static int insertSearch(int[] arr, int left, int right, int value) {
    System.out.println("----差值查找----");
    if (left > right || value < arr[0] || value > arr[arr.length -1]) {
        //如果需要查找的值，比最小还小或者是比最大还大，那么直接返回
        return -1;
    }

    //这是获取下标
    int middleIndex = left + (right - left) * (value - arr[left]) / (arr[right] - arr[left]);
    int middleValue = arr[middleIndex];

    if (value > middleValue) {
        //往右边进行查找
        return insertSearch(arr,left + 1,right,value);
    }else if (value < middleValue) {
        //往左边进行查找
        return insertSearch(arr,left,right - 1,value);
    }else {
        //查找到这个值
        return middleIndex;
    }
}
```



