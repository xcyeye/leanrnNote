---
date: 2022/3/25
---

类名::实例方法

对象::实例方法

类名::静态方法



# Lambda表达式学习

Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

使用 Lambda 表达式可以使代码变的更加简洁紧凑。

他的语法为

```java
(parameters) -> expression
or
(parameters) ->{ statements; }
```



重要特性：

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定表达式返回了一个数值。



## 函数式接口

在使用之前，我们必须的知道什么是函数式接口

函数式接口(`Single Abstract Method`)，就是一个接口中有且仅有一个抽象方法(可以有默认方法，或者是静态方法和从Object继承过来的方法)，在jdk8之后，在此接口上添加`@FunctionalInterface`注解就表示该接口是一个函数式接口，也称这样的接口为`Mark`类型的接口，如下面这个自定义的函数式接口

```java
@FunctionalInterface
public interface Aurora<T> {
    T get();
}
```

> 但是的清楚，函数式接口只是一个特例，就算一个接口只有一个抽象方法，没有被`@FunctionalInterface`直接标记，我们同样也是可以使用`lambda`



## lambda

直接放例子

```java
public interface Runnable {
    public abstract void run();
}
```



如果创建一个`Thread`，则可以下面这样

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println(1);
    }
});
```

还可以这样，因为此`Runnable`接口就一个方法

```java
Thread thread = new Thread(() -> {
    System.out.println("running");
});
```





## 使用

这里定义一个接口来使用

```java
public interface Aurora<T> {
    T get();
}
```

这个接口中，就只有一个`get`方法，没有参数，返回一个对象，然后我们就可以像下面这样使用了

```java
Aurora<Integer> aurora = () -> new Integer(12);
```

因为这里返回的是一个`Interger`对象，所以还可以这样

```java
Aurora<Integer> aurora = () -> 12;
```

如果接口中的`get`方法需要传入参数的话

```java
public interface Aurora<T> {
    T get(String s);
}

Aurora<String> aurora = t -> t;
System.out.println(aurora.get("a"));
System.out.println(aurora.get("b"));
System.out.println(aurora.get("c"));
```

运行结果

```java
a
b
c
```

> `Aurora<String> aurora = t -> t;`，因为`Aurora`接口中，get方法返回一个泛型，我们这里是返回一个String，get方法需要传入一个参数，所以这里参数我们使用`t`表示，因为t就是一个String，所以直接返回



需要注意的是，如果有返回值，我们需要实例化这个接口对象之后，手动调用，有参数的话，在进行赋值操作



## 案例

1. 如果薪资小于10000，涨工资到10000

    ```java
    public static void main(String[] args) {
        HashMap<String,Double> map = new HashMap<>();
        map.put("周瑜",9000.0);
        map.put("宋江",12000.0);
        map.put("张飞",8000.0);
    
        map.forEach((k,v) -> {
            if (v < 10000.0)
                map.put(k,10000.0);
        });//BiConsumer<T,U>，void apply(T t, U u)
        map.forEach((k,v) -> System.out.print(k + ":" + v + "\t\t"));
    }
    ```

2. ```java
    public static void main(String[] args) {
        //        Supplier<Student> s = () -> new Student();
    
        Supplier<Student> s = Student::new;
    }//实际过程：将new Student()赋值给了Supplier这个函数式接口中的那个抽象方法
    ```

3. 直接看这篇文章，更详细[深入理解Java Lambda表达式 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/365505945)