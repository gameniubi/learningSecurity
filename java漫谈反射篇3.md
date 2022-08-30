# java漫谈反射篇

## 前言

在上一篇文章中，我们了解到我们可以使用单例类中的get方法获得到实例对象。

但如果既没有公开的构造方法也没有如单例类中的get方法又应如何获得一个实例对象

如果一个对象的构造方法是私有的我们又是否可以调用他

## getconstruct

关于第一个问题，我们可以使用getconstruct来解决。
和getMethod 类似， getConstructor 接收的参数是构造函数列表类型，因为构造函数也支持重载，所以必须用参数列表类型才能唯一确定一个构造函数。

在获得到构造函数后，我们可以使用newinstance执行

```java
Class clazz = Class.forName("java.lang.ProcessBuilder");
((ProcessBuilder)clazz.getConstructor(List.class).newInstance(Arrays.asList("calc.exe"))).start();
```

我们通过传入list。class确定了唯一的构造方法，又利用newInstance传入数组值并执行

但在这段代码中，我们利用了强制转换，但在漏洞环境中是没有这种语法的。所以我们换一种利用方式

```java
Class clazz = Class.forName("java.lang.ProcessBuilder");
clazz.forName("start").invoke(clazz.getConstruct(List.class).newInstance(Arrays.asList("calc.exe")))
```

## getDeclaredMethod

getDeclareMethod方法可以解决第二个问题，与getmethod方法不同的是getDeclaredMethod方法是获得当前类声明的属性而非继承自父类的

具体的使用方法我们可以看一段代码示例

```java
Class clazz = Class.forName("java.lang.Runtime");
Constructor m = clazz.getDeclaredConstructor();
m.setAccessible(true);
clazz.getMethod("exec", String.class).invoke(m.newInstance(), "calc.exe");
```

setAccessible的作用是更改其作用域，否则这个私有方法依旧不可调用