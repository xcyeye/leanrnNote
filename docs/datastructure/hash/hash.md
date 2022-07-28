---
date: "2021/11/1 12:00"
---



# 哈希表

- 例题

::: tip

一个公司,当有新的员工来报道时,要求将该员工的信息加入(id,性别,年龄,住址..),当输入该员工的id时,要求查找到该员工的 所有信息.

> 要求: 不使用数据库,尽量节省内存,速度越快越好=>哈希表(散列) 

:::



## 概念

`散列表（Hash table，也叫哈希表）`，是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射(`例如id`)到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。

<img src="https://picture.xcye.xyz/image-20211101140649005.png?x-oss-process=style/pictureProcess1" alt="image-20211101140649005" style="zoom:50%;" />



::: tip

理解

哈希表就是在一个数组中，每一个下标对应的是一个链表(`如单链表`)，因为链表是通过指针进行两两之间的链接的，所以我们可以将一个节点，加入到指定的下标对应的那条链表上，但是一般都是通过取模运行，获取对应的下标

这里使用一个节点进行演示

```java 
public String name;
public int id;

/** 指向下一个节点 **/
public EmpNode next;
```

那么如果我们需要在这个哈希表上，添加新的节点，就可以通过`id % arr.length`得到此节点最终添加到哪个下标上

:::



## 代码演示



::: details code

```java
package vin.cco.hash;

import java.util.Scanner;

/**
 * 这是一个哈希表的代码演示
 * @author Administrator 程钦义
 * @blog https://blog.cco.vin
 * @date 2021/11/01 13:12
 **/

public class HashTabDemo {
    public static void main(String[] args) {
        HashTable hashTable = new HashTable(5);

        String key = "";
        Scanner scanner = new Scanner(System.in);
        while(true) {
            System.out.println("add:  a");
            System.out.println("list: l");
            System.out.println("find: f");
            System.out.println("exit: e");

            key = scanner.next();
            switch (key) {
                case "a":
                    System.out.println("id");
                    int id = scanner.nextInt();
                    System.out.println("name");
                    String name = scanner.next();
                    EmpNode emp = new EmpNode(name,id);
                    hashTable.add(emp);
                    break;
                case "l":
                    hashTable.show();
                    break;
                case "f":
                    System.out.println("id");
                    id = scanner.nextInt();
                    //hashTable.findEmpById(id);
                    break;
                case "e":
                    scanner.close();
                    System.exit(0);
                default:
                    break;
            }
        }
    }
}

/**
 * 这是一个哈希数组链表
 * */
class HashTable {
    /** 哈希表的最大长度 **/
    public int maxSize;

    /** 这是一个哈希数组 **/
    public EmpLinkList[] empLinkLists;


    public HashTable(int maxSize) {
        this.maxSize = maxSize;
        empLinkLists = new EmpLinkList[maxSize];

        for (int i = 0; i < maxSize; i++) {
            empLinkLists[i] = new EmpLinkList();
        }
    }

    public void add(EmpNode empNode) {
        empLinkLists[hashFun(empNode)].add(empNode);
    }

    public void show() {
        for (int i = 0; i < empLinkLists.length; i++) {
            empLinkLists[i].show(i);
        }
    }

    private int hashFun(EmpNode empNode) {
        return empNode.id % maxSize;
    }
}

/**
 * 这是一个员工数据的节点
 * */
class EmpNode {

    /** 姓名 **/
    public String name;
    public int id;

    /** 指向下一个节点 **/
    public EmpNode next;

    public EmpNode(String name, int id) {
        this.name = name;
        this.id = id;
    }
}

/**
 * 这是一个单链表
 * */
class EmpLinkList {
    /** 这是一个头指针 **/
    public EmpNode head;

    public void add(EmpNode empNode) {
        //向链表中，插入节点
        if (head == null) {
            head = empNode;
            return;
        }

        EmpNode tempNode = head;
        //链表不为空
        while (true) {
            if (tempNode.next == null) {
                break;
            }

            tempNode = tempNode.next;
        }

        //找到最后一个节点
        tempNode.next = empNode;
    }

    public void show(int id) {
        //遍历链表
        EmpNode temNode = head;
        if (temNode == null) {
            System.out.println(id + "号链表为空");
            return;
        }

        while (true) {
            System.out.printf(id + "号链表信息: ");
            System.out.printf("===> {"+ temNode.name + "," + temNode.id +"}");

            if (temNode.next == null) {
                break;
            }
            temNode = temNode.next;
        }

        System.out.println();
    }
}
```

:::



类结构

```java
//包含主方法
public class HashTabDemo {
  
}

/**
 * 这是一个哈希数组链表，也就是一个数组
 * */
class HashTable {
  
}

/**
 * 节点
 * */
class EmpNode {

}

/**
 * 这是一个单链表，此单链表中的每一个节点，都是EmpNode对象
 * */
class EmpLinkList {
    
}
```

::: tip

其中，HashTable是一个数组，我们最终操作的也是这个对象，此对象中，存在诸如add,list,find,delete等删除节点的方法，这些方法会再次调用EmpLinkList对象中对应的方法，对哈希表进行增加，删除等

所以最终的方法执行都是在EmpLinkList对象中

:::



## 总结

哈希表因为使用了数组的方式，所以其在数据查找和遍历方面有较大的优势(`如果是一个有序数组，还可以配置二分查找，查找速度更快`)，但是对于数组的长度不够，需要扩容，或者新增一个链表，删除一个链表的时候，会对数组的下标进行移动，会特别耗费时间，这是他的逆势，所以就有了`树`的诞生，请查看[二叉树](../tree/binarytree.md)



