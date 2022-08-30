# JAVA漫谈笔记--反射篇（一）

## 前言

	Java安全可以从反序列列化漏漏洞洞开始说起，反序列列化漏漏洞洞⼜又可以从反射开始说起。
	
	反射是⼤大多数语⾔言⾥里里都必不不可少的组成部分，对象可以通过反射获取他的类，类可以通过反射拿到所有

方法（包括私有），拿到的⽅方法可以调⽤用，总之通过“反射”，我们可以将Java这种静态语⾔言附加上动态

特性。

	这半年年我⽼提到“动态特性”，我给这四个字赋予了了⼀一个含义：“⼀一段代码，改变其中的变量量，将会导致

这段代码产⽣生功能性的变化，我称之为动态特性”。

## Class类与其常用方法

	Class 类是在Java语言中定义一个特定类的实现。一个类的定义包含成员变量，成员方法，还有这个类实现的接口，以及这个类的父类。Class类的对象用于表示当前运行的 Java 应用程序中的类和接口。

```java
forname()

//通过名字获取类
  
getClassLoader()

//获取该类的类装载器。

getComponentType()

//如果当前类表示一个数组，则返回表示该数组组件的Class对象，否则返回null。

getConstructor(Class[])

//返回当前Class对象表示的类的指定的公有构造子对象。

getConstructors()

//返回当前Class对象表示的类的所有公有构造子对象数组。

getDeclaredConstructor(Class[])

//返回当前Class对象表示的类的指定已说明的一个构造子对象。

getDeclaredConstructors()

//返回当前Class对象表示的类的所有已说明的构造子对象数组。

getDeclaredField(String)

//返回当前Class对象表示的类或接口的指定已说明的一个域对象。

getDeclaredFields()

//返回当前Class对象表示的类或接口的所有已说明的域对象数组。

getDeclaredMethod(String,Class[])

//返回当前Class对象表示的类或接口的指定已说明的一个方法对象。

getDeclaredMethods()

//返回Class对象表示的类或接口的所有已说明的方法数组。

getField(String)

//返回当前Class对象表示的类或接口的指定的公有成员域对象。

getFields()

//返回当前Class对象表示的类或接口的所有可访问的公有域对象数组。

getInterfaces()

//返回当前对象表示的类或接口实现的接口。

getMethod(String,Class[])

//返回当前Class对象表示的类或接口的指定的公有成员方法对象。

getMethods()

//返回当前Class对象表示的类或接口的所有公有成员方法对象数组，包括已声明的和从父类继承的方法。

getModifiers()

//返回该类或接口的Java语言修改器代码。

getName()

//返回Class对象表示的类型(类、接口、数组或基类型)的完整路径名字符串。

getResource(String)

//按指定名查找资源。

getResourceAsStream(String)

//用给定名查找资源。

getSigners()

//获取类标记。

getSuperclass()

//如果此对象表示除Object外的任一类,那么返回此对象的父类对象。

isArray()

//如果Class对象表示一个数组则返回true,否则返回false。

isAssignableFrom(Class)

//判定Class对象表示的类或接口是否同参数指定的Class表示的类或接口相同，或是其父类。

isInstance(Object)

//此方法是Java语言instanceof操作的动态等价方法。

isInterface()

//判定指定的Class对象是否表示一个接口类型。

isPrimitive()

//判定指定的Class对象是否表示一个Java的基类型。

newInstance()

//创建类的新实例。

toString()

//将对象转换为字符串。

```

## forName方法与其两个重载

~~~java
Class<?> forName(String name)
Class<?> forName(String name, **boolean** initialize, ClassLoader loader)
~~~

默认情况下， forName 的第⼀一个参数是类名；第⼆二个参数表示是否初始化；第三个参数就是ClassLoader 。

Classloader为类加载器，他会告诉虚拟机如何加载该类[详细的类加载器解读](https://www.cnblogs.com/yepei/p/5649263.html)

### initialize的解读

initialize为一个布尔值，该值决定了是否初始化类

### 关于初始化

我们看如下一段代码

~~~java
public class TrainPrint {
	{
		System.out.printf("Empty block initial %s\n", this.getClass());
	}
	static {
		System.out.printf("Static initial %s\n", TrainPrint.class);
	}
	public TrainPrint() {
	System.out.printf("Initial %s\n", this.getClass());
	}
}
~~~



其中存在三个方法，分别为代码块、静态代码块、构造函数

在类初始化时，会调用静态代码块。而代码块方法会放在@super与此类构造函数代码之间。

### 是否初始化

在调用forName()方法时，若initialize为默认值true会初始化类，而使用.class时不会初始化类