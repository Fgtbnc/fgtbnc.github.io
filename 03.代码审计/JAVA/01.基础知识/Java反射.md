---
title: Java反射
date: 2023/5/1 15:00:25
categories:
- Java
---

# 反射

## 概念

> 反射是Java的特征之一，是一种间接操作目标对象的机制，核心是**JVM在运行状态的时候才动态加载类**，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。通过使用反射我们不仅可以获取到任何类的成员方法(Methods)、成员变量(Fields)、构造方法(Constructors)等信息，还可以动态创建Java类实例、调用任意的类方法、修改任意的类成员变量值等。

类加载流程如下

![image-20230502220543579](../images/EsePjI7n9CtyqbA.png)

JVM在第一次读取到一种`class`类型时，将其加载进内存后会为其创建一个`Class`类型的实例，并在该实例中保存了该`class`的所有信息，包括类名、包名、父类、实现的接口、所有方法、字段等，因此，如果获取了某个`Class`实例，我们就可以通过这个`Class`实例获取到该实例对应的`class`的所有信息。



## 相关类

```java
java.lang.Class //类对象

java.lang.reflect.Constructor //类的构造器对象

java.lang.reflect.Field //类的属性对象

java.lang.reflect.Method //类的方法对象

java.lang.reflect.Modifier //类的修饰符对象
```



## 使用

`Class`类型实际上是一个名叫`Class`的`class`

```java
public final class Class {
    private Class() {}
}
```

可以看出Class类的构造器是私有的，所以只有JVM能够创建Class对象。

如果我们想使用就需要以下几种方法获得：



### 获得Class类

```java
//调用Class类的静态方法forName获取某个类的Class类实例
Class clz = Class.forName("User");
注：类名默认为类完整路径,如java.lang.Runtime,通常使用这个来获得Class类实例

//调用某个类实例的getClass()方法
Class clz = (new User()).getClass();    
    
    
//访问某个类实例的class属性，这个属性就存储着这个类对应的Class类的实例
Class clz = (new User()).class;
```

### 获得构造函数

```java
Constructor<?>[] getConstructors() ：只返回public构造函数

Constructor<?>[] getDeclaredConstructors() ：返回所有构造函数

Constructor<> getConstructor(类<?>... parameterTypes) : 匹配和参数配型相符的public构造函数

Constructor<> getDeclaredConstructor(类<?>... parameterTypes) ： 匹配和参数配型相符的构造函数
```



### 实例化类

```java
clz.newInstance();
```

> 实际上是调用了该类的无参构造函数
>
> 所以前提条件：
>
> 该类有无参构造函数，并且是公有的

### 获得成员方法

```java
Method method=Class.getMethod(String name, 类<?>... parameterTypes) //返回该类所声明的public方法

Method method=Class.getDeclaredMethod(String name, 类<?>... parameterTypes) //返回该类所声明的所有方法

//第一个参数为方法名，第二个参数为这个方法的参数类型

    
Method method=Class.getMethods() //获取所有的public方法，包括类自身声明的public方法，父类中的public方法、实现的接口方法

Method method=Class.getDeclaredMethods() // 获取该类中的所有方法
```



### 使用成员方法

```java
method.invoke(类实例, [该方法的参数类型]);
```

> **如果调用这个方法是普通方法，第一个参数就是类对象；**
>
> **如果调用这个方法是静态方法，第一个参数就是类；**



### 获取成员变量

```java
Field[] getFields() ：获取所有 public 修饰的成员变量

Field[] getDeclaredFields() 获取所有的成员变量，不考虑修饰符

Field getField(String name) 获取指定名称的 public 修饰的成员变量

Field getDeclaredField(String name) 获取指定的成员变量
```



## 实例

常用的RCE

```java
Runtime.getRuntime().exec("calc");
```

> 注意`Runtime`这个类的构造函数是私有的，所以我们正常只能通过公有的方法`Runtime.getRuntime() `来获取到 Runtime 对象

通过反射实现上述代码

```java
import java.lang.reflect.Method;

//首先需要获得Runtime类的Class实例
Class clz = Class.forName("java.lang.Runtime");
//获得Runtime类的getRuntime方法
Method getruntime = clz.getMethod("getRuntime");

//调用该方法获得Runtime对象
Object runtime = getruntime.invoke(clz);
//获得Runtime类的exec方法
Method exec = clz.getMethod("exec",String.class);
//调用exec方法执行命令
exec.invoke(runtime,"calc");
```

整合成一句

```java
Class.forName("java.lang.Runtime").getMethod("exec", String.class).invoke(Class.forName("java.lang.Runtime").getMethod("getRuntime").invoke(Class.forName("java.lang.Runtime")), "calc.exe");
```

### 强制访问私有，保护

```java
setAccessible(true);
```

上述命令执行代码可以转为

```java
import java.lang.reflect.Constructor;

public class RuntimeTest {
    public static void main(String[] args) throws Exception {
        Class c1= Class.forName("java.lang.Runtime");
        
        // 获得构造函数
        Constructor m = c1.getDeclaredConstructor();
        // 获得访问私有方法和属性的权限
        m.setAccessible(true);
        
        c1.getMethod("exec", String.class).invoke(m.newInstance(), "calc");
    }
}
```



### 修改`final`字段

![image-20230503115304804](../images/image-20230503115304804.png)

- 只有final修饰，并且为间接赋值，可以修改

- 如果是final+static修饰，并且为间接赋值，需要先去掉final修饰符才能修改

  ```java
  // 获得modifiers
  Field nameModifyField = nameField.getClass().getDeclaredField("modifiers");
  // 修改权限
  nameModifyField.setAccessible(true);
  // 去除final修饰符
  nameModifyField.setInt(nameField, nameField.getModifiers() & ~Modifier.FINAL);
  // 修改Field值
  nameField.set(m,new StringBuilder("Drunkbaby Too Silly"));
  // 重新添加上final修饰符
  nameModifyField.setInt(nameField, nameField.getModifiers() & ~Modifier.FINAL);
  ```

