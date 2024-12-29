---
title: Java泛型
date: 2023/5/1 15:00:25
categories:
- Java
---

# 泛型

[java 泛型详解-绝对是对泛型方法讲解最详细的，没有之一 - little fat - 博客园 (cnblogs.com)](https://www.cnblogs.com/coprince/p/8603492.html)

## 介绍

```java
List<String> stringList = new ArrayList<String>(); // 使用泛型类List，并传入String类型实参
stringList.add("Hello");
stringList.add("World");
String firstElement = stringList.get(0); // 获取列表中的第一个元素
```

> 定义一个泛型类`List<T>`，其中`T`为类型形参，表示列表中元素的类型。在使用该泛型类时，需要传入具体的类型实参作为类型形参的取值。

- **类型形参**指的是在定义泛型类、接口或方法时使用的占位符类型。它们用于参数列表、返回值类型、局部变量等位置，用来表示具体的数据类型还未确定。通常使用大写字母来表示类型形参，例如`T`、`E`、`K`等。

- **类型实参**指的是在使用泛型类、接口或方法时实际传入的具体数据类型。它们用于替换类型形参，使泛型类、接口或方法具体化。

  ![img](../images/java.lang.png?lastModify=1695619823)





## 泛型的作用

**代码复用**

![image-20230925132013782](../images/image-20230925132013782.png)



**类型安全，编译器会检查类型**

![image-20230925130137491](../images/image-20230925130137491.png)

引入泛型后，它将提供类型的约束，提供编译前的检查：

![image-20230925130207681](../images/image-20230925130207681.png)





## 泛型类

```java
//此处T可以随便写为任意标识，常见的如<T> Type <E> Element <K> Key <V> value
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{
    //key这个成员变量的类型为T,T的类型由外部指定
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }

    public static void main(String[] args) {
        //泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
		//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
        Generic<Integer> genericInteger = new Generic<Integer>(123456);

		//传入的实参类型需与泛型的类型参数类型相同，即为String.
        Generic<String> genericString = new Generic<String>("key_vlaue");
        System.out.println("泛型测试 key is " + genericInteger.getKey());
        System.out.println("泛型测试 key is " + genericString.getKey());
    }
}
```

![image-20230925131006945](../images/image-20230925131006945.png)



## 泛型接口

```java
//定义一个泛型接口
public interface Generator<T> {
    public T next();
}
```

```java
/**
 * 传入泛型实参时：
 * 定义一个生产器实现这个接口,虽然我们只创建了一个泛型接口Generator<T>
 * 但是我们可以为T传入无数个实参，形成无数种类型的Generator接口。
 * 在实现类实现泛型接口时，如已将泛型类型传入实参类型，则所有使用泛型的地方都要替换成传入的实参类型
 * 即：Generator<T>，public T next();中的的T都要替换成传入的String类型。
 */
public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```



## 泛型通配符

`Ingeter`是`Number`的一个子类，但是`Generic<Integer>`不能被看作为`Generic<Number>`的子类。

![image-20230925134703767](../images/image-20230925134703767.png)

由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

因此我们需要一个在逻辑上可以表示同时是`Generic<Integer>`和`Generic<Number>`父类的引用类型。由此类型通配符`?`应运而生。

![image-20230925134758827](../images/image-20230925134758827.png)

**类型通配符一般是使用？代替具体的类型实参，此处的？和Number、String、Integer一样都是一种实际的类型，可以把？看成所有类型的父类。是一种真实的类型。**



## 泛型方法

**泛型类中的使用了泛型的成员方法并不是泛型方法**

```java
public class Generic<T>{
    public Generic(T key) {
        this.key = key;
    }
}
```

该泛型类的构造方法形参key的类型也为T，但是这个方法并不是泛型方法，只是类中一个普通的成员方法。因为该方法能使用T是因为在声明泛型类时T已经被指定了。

如果该方法使用了泛型类没有声明的泛型E，编译器就会报错

![image-20230925155644795](../images/image-20230925155644795.png)

**只有声明了`<>`的方法才是泛型方法**

```java
public class Generic<T>{
    public <E> T getKey(){
        return key;
    }
}
```

