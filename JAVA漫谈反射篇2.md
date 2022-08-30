# JAVA漫谈 反射篇(二)

## 通过forName加载内部类

java支持在普通类中编写内部类，我们可以将其当做两个不同的类看待，通过$加载内部类

如类c1中存在内部类c2，我们就可以通过Class.forName(c1$c2)来获得c2的class对象

获得类后我们可以继续使用反射来获得这个类的更多信息，如方法、成员变量等

## newInstance()的细节

class.newinstance这个方法用来调用无参构造函数实例化一个对象，但有时我们发现无法成功实例化对象。这有可能是因为其没有无参构造函数或构造函数私有化。



我们使用Runtime类举例

Runtime是典型的单例模式，其构造方法私有化，无法使用class.newInstance()实例化对象

```java
Class clazz = Class.forName("java.lang.Runtime");
clazz.getMethod("exec", String.class).invoke(clazz.newInstance(), "id");
```

该语句就会报错



这时我们需要调用其getRuntime()方法获得其实例对象

```java
Class clazz = Class.forName("java.lang.Runtime");
Runtime getRuntime = (Runtime) clazz.getMethod("getRuntime").invoke(clazz);
clazz.getMethod("exec", String.class).invoke(getRuntime,"mkdir aaa");
```



## 补充：单例模式

在某类只需要实例化一次时，采用单例模式。如下就是一个单例模式的类。

```java
public class Hello {
    private static Hello hello=new Hello;

    private Hello() {
        return hello;
    }

    public static Hello getHello() {
        return hello;
    }
}

```

在单例模式中，该类只会被实例化一次。之后所有的调用都会调用同一个对象

