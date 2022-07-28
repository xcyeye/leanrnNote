---
date: "2021/22/2 8:23"
---





# 二叉树

> 为什么需要树这种数据结构？

- 数组存储方式

  > 优点: 通过下标方式访问元素，速度快。对于有序数组，还可使用二分查找提高检索速度。
  >
  > 缺点：如果要检索具体某个值，或者插入值(按一定顺序)会整体移动，效率较低

- 链式存储方式

  > 优点：在一定程度上对数组存储方式有优化(比如：插入一个数值节点，只需要将插入节点，链接到链表中即可， 删除效率也很好)。
  >
  > 缺点：在进行检索时，效率仍然较低，比如(检索某个值，需要从头节点开始遍历) 

- 树存储方式

  > 能提高数据存储，读取的效率,  比如利用 二叉排序树(Binary Sort Tree)，既可以保证数据的检索速度，同时也可以保证数据的插入，删除，修改的速度。





## 概念

- 树有很多种，每个节点最多只能有两个子节点的一种形式称为`二叉树`。
- 二叉树的子节点分为`左节点`和`右节点`。

![](https://picture.xcye.xyz/image-20211101184352949.png?x-oss-process=style/pictureProcess1)



- 如果该二叉树的所有叶子节点都在最后一层，并且结点总数= `2^n -1`, n 为层数，则我们称为满二叉树。
- 如果该二叉树的所有叶子节点都在最后一层或者倒数第二层，而且最后一层的叶子节点在左边连续，倒数第二层的叶子节点在右边连续，我们称为完全二叉树

![](https://picture.xcye.xyz/image-20211101184500024.png?x-oss-process=style/pictureProcess1)







## 代码实现

首先需要确定根节点，对于其中的每一个子节点都可能会存在一个`left`指针和`right`指针

`节点`

```java
/**
 * 这是一个节点
 * @author qsyyke
 * @blog https://blog.cco.vin
 * @date 2021/11/1
 **/
class HeroNode {
    private int id;
    private String name;
    /** 左节点 **/
    private HeroNode left;
    /** 右节点 **/
    private HeroNode right;

    public HeroNode(int id, String name) {
        this.id = id;
        this.name = name;
    }

    /**
     * 这是一个二叉树的前序遍历
     */
    public void preList() {
        //因为前序遍历都是从父节点开始，所以可以先输出父节点
        System.out.println(this);

        if (this.left != null) {
            this.left.preList();
        }
        if (this.right != null) {
            this.right.preList();
        }
    }

    /**
     * 这是一个二叉树的中序遍历
     */
    public void middleList() {
        //因为中序遍历，父节点是中间的时候输出
        if (this.left != null) {
            this.left.middleList();
        }

        System.out.println(this);
        if (this.right != null) {
            this.right.middleList();
        }
    }

    /**
     * 这是一个二叉树的后序遍历
     */
    public void nextList() {
        //因为后续遍历，父节点是最后才输出的
        if (this.left != null) {
            this.left.nextList();
        }
        if (this.right != null) {
            this.right.nextList();
        }

        System.out.println(this);
    }


    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public HeroNode getLeft() {
        return left;
    }

    public void setLeft(HeroNode left) {
        this.left = left;
    }

    public HeroNode getRight() {
        return right;
    }

    public void setRight(HeroNode right) {
        this.right = right;
    }

    @Override
    public String toString() {
        return "HeroNode{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}
```

`二叉树`

```java
/**
 * 这是一个二叉树
 * @author qsyyke
 * @blog https://blog.cco.vin
 * @date 2021/11/1
 **/
class BinaryTree {
    private HeroNode root;

    public BinaryTree(HeroNode root) {
        this.root = root;
    }

    public HeroNode getRoot() {
        return root;
    }

    public void setRoot(HeroNode root) {
        this.root = root;
    }

    public void preList() {
        this.root.preList();
    }
    public void middleList() {
        this.root.middleList();
    }
    public void nextList() {
        this.root.nextList();
    }
}
```



### demo

构建如下图所示的二叉树

![](https://picture.xcye.xyz/image-20211101185925888.png?x-oss-process=style/pictureProcess1)

127896 10 3 5 4 11

8 7 9 2 6 10 1 5 3 4 11

8 9 7 10 6 2 5 11 4 2 1



`demo`

```java
public class BinaryTreeDemo {
    public static void main(String[] args) {
        HeroNode root = new HeroNode(1,"宋江");
        HeroNode node2 = new HeroNode(2,"吴用");
        HeroNode node3 = new HeroNode(3,"卢俊义");
        HeroNode node4 = new HeroNode(4,"林冲");
        HeroNode node5 = new HeroNode(5,"关胜");
        HeroNode node6 = new HeroNode(6,"6吴用");
        HeroNode node7 = new HeroNode(7,"7吴用");
        HeroNode node8 = new HeroNode(8,"8吴用");
        HeroNode node9 = new HeroNode(9,"9吴用");
        HeroNode node10 = new HeroNode(10,"10吴用");
        HeroNode node11 = new HeroNode(11,"11吴用");

        //手动构建二叉树
        root.setLeft(node2);
        node2.setLeft(node7);
        node2.setRight(node6);
        node6.setRight(node10);
        node7.setLeft(node8);
        node7.setRight(node9);
        root.setRight(node3);
        node3.setRight(node4);
        node3.setLeft(node5);
        node4.setRight(node11);

        System.out.println("前序遍历");
        root.preList(); //1,2,3,4

        System.out.println("中序遍历");
        root.middleList();

        System.out.println("后续遍历");
        root.nextList();

    }
}
```

那么其输出结果为

::: details view

```java
前序遍历
HeroNode{id=1, name='宋江'}
HeroNode{id=2, name='吴用'}
HeroNode{id=7, name='7吴用'}
HeroNode{id=8, name='8吴用'}
HeroNode{id=9, name='9吴用'}
HeroNode{id=6, name='6吴用'}
HeroNode{id=10, name='10吴用'}
HeroNode{id=3, name='卢俊义'}
HeroNode{id=5, name='关胜'}
HeroNode{id=4, name='林冲'}
HeroNode{id=11, name='11吴用'}
中序遍历
HeroNode{id=8, name='8吴用'}
HeroNode{id=7, name='7吴用'}
HeroNode{id=9, name='9吴用'}
HeroNode{id=2, name='吴用'}
HeroNode{id=6, name='6吴用'}
HeroNode{id=10, name='10吴用'}
HeroNode{id=1, name='宋江'}
HeroNode{id=5, name='关胜'}
HeroNode{id=3, name='卢俊义'}
HeroNode{id=4, name='林冲'}
HeroNode{id=11, name='11吴用'}
后续遍历
HeroNode{id=8, name='8吴用'}
HeroNode{id=9, name='9吴用'}
HeroNode{id=7, name='7吴用'}
HeroNode{id=10, name='10吴用'}
HeroNode{id=6, name='6吴用'}
HeroNode{id=2, name='吴用'}
HeroNode{id=5, name='关胜'}
HeroNode{id=11, name='11吴用'}
HeroNode{id=4, name='林冲'}
HeroNode{id=3, name='卢俊义'}
HeroNode{id=1, name='宋江'}
```

:::







::: tip

代码实现部分，需要注意的是，所有的遍历操作都是在节点中进行的，都是从`root`节点开始，



:::


## 遍历

### 前序遍历

```java
public void preList() {
    //因为前序遍历都是从父节点开始，所以可以先输出父节点
    System.out.println(this);

    if (this.left != null) {
        this.left.preList();
    }
    if (this.right != null) {
        this.right.preList();
    }
}
```





### 中序遍历

```java
public void middleList() {
    //因为中序遍历，父节点是中间的时候输出
    if (this.left != null) {
        this.left.middleList();
    }

    System.out.println(this);
    if (this.right != null) {
        this.right.middleList();
    }
}
```







### 后序遍历

```java
public void nextList() {
    //因为后续遍历，父节点是最后才输出的
    if (this.left != null) {
        this.left.nextList();
    }
    if (this.right != null) {
        this.right.nextList();
    }

    System.out.println(this);
}
```





## 查找

使用二叉树的方式进行某个节点的查找，并比较三个查找方法的效率(`前序查找，中序查找，后序查找`)



### 前序查找

```java
public HeroNode preSearch(int id) {

    //因为前序查找，最开始都是和父节点进行比较的，所以首先应该和父节点进行比较，并且在之后的递归中，最终都是
    //和这个this.id进行比较的
    System.out.println("进行前序查找");
    if (this.id == id) {
        return this;
    }

    //如果执行到这里，说明需要对左右节点进行查找
    HeroNode tempNode = null;
    if (this.left != null) {
        tempNode = this.left.preSearch(id);
    }

    if (tempNode != null) {
        //说明在左节点中，找到了该id对应的节点
        return tempNode;
    }

    //继续对右节点进行查找
    if (this.right != null) {
        tempNode = this.right.preSearch(id);
    }

    return tempNode;
}
```



### 中序查找

```java
public HeroNode middleSearch(int id) {

    //判断是否和左节点的id相同
    HeroNode tempNode = null;
    if (this.left != null) {
        tempNode = this.left.middleSearch(id);
    }

    if (tempNode != null) {
        return tempNode;
    }

    System.out.println("进行中序查找");
    if (this.id == id) {
        return this;
    }

    if (this.right != null) {
        tempNode = this.right.middleSearch(id);
    }

    return tempNode;
}
```



### 后序查找

```java
public HeroNode nextSearch(int id) {
    HeroNode tempNode = null;
    if (this.left != null) {
        tempNode = this.left.nextSearch(id);
    }

    if (tempNode != null) {
        return tempNode;
    }

    if (this.right != null) {
        tempNode = this.right.nextSearch(id);
    }

    if (tempNode != null) {
        return tempNode;
    }

    System.out.println("进行后续查找");
    if (this.id == id) {
        return this;
    }

    return null;
}
```



::: tip

在进行各查询方法执行次数的比较时，计算次数都需要在`if (this.id == id) {`的前一行进行，因为无论是哪种查找，他们最终进行id比较，都是在此if语句中

:::

### 比较

通过对同一个节点的不同查找方法，发现后续查找执行的最短

```java
进行前序查找----------
进行前序查找
进行前序查找
进行前序查找
进行前序查找
进行前序查找
HeroNode{id=9, name='9吴用'}
进行中序查找---------
进行中序查找
进行中序查找
进行中序查找
HeroNode{id=9, name='9吴用'}
进行后续查号-------
进行后续查找
进行后续查找
HeroNode{id=9, name='9吴用'}
```





## 删除节点

1. 如果删除的是叶子节点，则直接删除该节点
2. 如果删除的不是叶子节点，是子树，那么删除该子树



```java
//BinaryTree.java
public void delNode(int id) {
    if (this.root.getId() == id) {
        //该根节点就是待删除的节点
        this.root = null;
        return;
    }
    this.root.delNode(id);
}
```

```java
//HeroNode.java
//节点的删除 使用后序删除节点的方式，因为后序能够更快的找到id所匹配的节点
public void delNode(int id) {
    HeroNode tempNode = null;
    if (this.left != null) {
        if (this.left.id == id) {
            this.left = null;
            return;
        }
    }

    if (this.right != null) {
        if (this.right.id == id) {
            this.right = null;
            return;
        }
    }
    //运行到这里，说明此this的左右节点都不是需要删除的节点，则继续使用递归的方式进行查找
    if (this.left != null) {
        this.left.delNode(id);
    }

    if (this.right != null) {
        this.right.delNode(id);
    }
}
```

:::  tip

删除节点的时候，我们都是判断当前节点的左右节点是否为待删除的节点，并不是判断当前节点是不是待删除的节点，所以就不需要在`if (this.right.id == id) {`后加上`if (this.id == id) {`，因为判断当前节点的左右子节点是否是待删除的节点，此this节点就已经在此递归的上一步就进行判断了，就不必在此在进行判断

:::







