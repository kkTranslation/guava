# `TypeToken`

由于类型擦除，你不能够在运行时传递泛型类对象——你可能想强制转换它们，并假装这些对象是有泛型的，但实际上它们没有。

举例:

```java
ArrayList<String> stringList = Lists.newArrayList();
ArrayList<Integer> intList = Lists.newArrayList();
System.out.println(stringList.getClass().isAssignableFrom(intList.getClass()));
  // returns true, even though ArrayList<String> is not assignable from ArrayList<Integer>
```

**Guava provides**<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/TypeToken.html'><code>TypeToken</code></a>, 它使用基于反射的技巧，允许你操纵和查询通用类型，即使在运行时.`Think`的`TypeToken`作为一种创建，操作和查询类型（和隐式`Class`）对象的方法。

注意Guice用户：`TypeToken`类似于 [Guice](https://github.com/google/guice)的 [`TypeLiteral`](https://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/TypeLiteral.html)类，但有一个重要的区别：它支持非限定类型，如T，`List<T>`或 `List<? extends Number>`; 而TypeLiteral没有。 `TypeToken`也是可序列化的，并提供了许多其他实用程序方法。

### 背景：类型擦除和反射

Java在运行时不保留对象的通用类型信息。 如果在运行时有一个`ArrayList <String>`对象，你不能确定它有通用类型`ArrayList <String>` - 并且你可以用不安全的原始类型将它转换为`ArrayList <Object>`。

但是，反射允许您检测通用类型的方法和类。 如果实现一个返回`List <String>`的方法，并且使用反射来获得该方法的返回类型，您将返回一个表示`List <String>`的 <a href='http://docs.oracle.com/javase/6/docs/api/java/lang/reflect/ParameterizedType.html'><code>ParameterizedType</code></a> .

`TypeToken`类使用此解决方法允许以最小的语法开销操作泛型类型。

### 介绍
获取一个基本的，原始类`TypeToken`是简单的

```java

TypeToken<String> stringTok = TypeToken.of(String.class);
TypeToken<Integer> intTok = TypeToken.of(Integer.class);
```

为了获得`TypeToken`与泛型类型 - 当您在编译时知道泛型类型参数时，您将使用一个空的匿名内部类：

```java

TypeToken<List<String>> stringListTok = new TypeToken<List<String>>() {};
```

或者如果您想故意引用通配符类型：

```java

TypeToken<Map<?, ?>> wildMapTok = new TypeToken<Map<?, ?>>() {};
```

`TypeToken`提供了一种动态解析泛型类型参数的方式，如下所示：

```java

static <K, V> TypeToken<Map<K, V>> mapToken(TypeToken<K> keyToken, TypeToken<V> valueToken) {
  return new TypeToken<Map<K, V>>() {}
    .where(new TypeParameter<K>() {}, keyToken)
    .where(new TypeParameter<V>() {}, valueToken);
}
...
TypeToken<Map<String, BigInteger>> mapToken = mapToken(
   TypeToken.of(String.class),
   TypeToken.of(BigInteger.class));
TypeToken<Map<Integer, Queue<String>>> complexToken = mapToken(
   TypeToken.of(Integer.class),
   new TypeToken<Queue<String>>() {});
```

请注意，刚刚如果`mapToken`返回`new TypeToken<Map<K, V>>()`，它实际上不能重新分配给`K`和`V`的类型，所以例如

```java

class Util {
  static <K, V> TypeToken<Map<K, V>> incorrectMapToken() {
    return new TypeToken<Map<K, V>>() {};
  }
}

System.out.println(Util.<String, BigInteger>incorrectMapToken());
// just prints out "java.util.Map<K, V>"
```

或者，您可以捕获具有（通常是匿名）子类的泛型类型，并根据知道类型参数的上下文类来解析它。

```java

abstract class IKnowMyType<T> {
  TypeToken<T> type = new TypeToken<T>(getClass()) {};
}
...
new IKnowMyType<String>() {}.type; // 返回一个正确的TypeToken <String>
```

利用这种技术，你可以，例如，获取知道他们的元素类型的类。

## Queries

`TypeToken`支持`Class`所支持的许多查询，但是通用约束被恰当地考虑在内。

支持的查询操作包括：

| 方法                    | 描述                              |
| :------------------------ | :--------------------------------------- |
| `getType()`               | 返回包装的`java.lang.reflect.Type`。 |
| `getRawType()`            | 返回最著名的运行时类。    |
| `getSubtype(Class<?>)`    | 返回一些具有指定raw类的`this`子类型。 例如，如果这是`Iterable <String>`，参数是`List.class`，结果将是`List <String>`。 |
| ` getSupertype(Class<?>)` | 将指定的raw类生成为此类型的超类型。 例如，如果这是`Set <String>`，参数是`Iterable.class`，结果将是`Iterable <String>`。 |
| `isAssignableFrom(type)`  | 如果此类型可以从给定类型分配，则返回`true`，并考虑到通用参数。 `List<? extends Number>`可以从`List <Integer>`中分配，但`List <String>`不是。 |
| `getTypes()`              | 返回此类型为或类型的所有类和接口的集合。 返回的`Set`还提供了方法`classes()`和`interfaces()`，让你只查看超类和超级界面。 |
| `isArray()`               | 检查这个类型是否已知是一个数组，如`int []`甚至`<? extends A[]>`。 |
| `getComponentType()`      | 返回数组组件类型。        |

### `resolveType`

`resolveType`是一个功能强大而又复杂的查询操作，可用于从上下文标记中“substitute”类型参数。 例如，
```java

TypeToken<Function<Integer, String>> funToken = new TypeToken<Function<Integer, String>>() {};

TypeToken<?> funResultToken = funToken.resolveType(Function.class.getTypeParameters()[1]));
  // returns a TypeToken<String>
```

`TypeToken`将Java提供的`TypeVariable`与“context”标记中的那些类型变量的值相统一。 这可以用于一般地推导出类型的方法的返回类型：
```java

TypeToken<Map<String, Integer>> mapToken = new TypeToken<Map<String, Integer>>() {};
TypeToken<?> entrySetToken = mapToken.resolveType(Map.class.getMethod("entrySet").getGenericReturnType());
  // returns a TypeToken<Set<Map.Entry<String, Integer>>>
```


# `Invokable`

Guava <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/Invokable.html'><code>Invokable</code></a> is a fluent wrapper of java.lang.reflect.Method and java.lang.reflect.Constructor. It simplifies common reflective code using either. Some usage examples follow:

### `Is the method public?`

JDK:
```java

Modifier.isPublic(method.getModifiers())
```

Invokable:
```java

invokable.isPublic()
```

### `Is the method package private?`

JDK:
```java

!(Modifier.isPrivate(method.getModifiers()) || Modifier.isPublic(method.getModifiers()))
```

Invokable:
```java

invokable.isPackagePrivate()
```

### `Can the method be overridden by subclasses?`

JDK:
```java

!(Modifier.isFinal(method.getModifiers())
    || Modifiers.isPrivate(method.getModifiers())
    || Modifiers.isStatic(method.getModifiers())
    || Modifiers.isFinal(method.getDeclaringClass().getModifiers()))
```

Invokable:
```java

invokable.isOverridable()
```

### `Is the first parameter of the method annotated with @Nullable?`

JDK:
```java

for (Annotation annotation : method.getParameterAnnotations()[0]) {
  if (annotation instanceof Nullable) {
    return true;
  }
}
return false;
```

Invokable:
```java

invokable.getParameters().get(0).isAnnotationPresent(Nullable.class)
```

### `How to share the same code for both constructors and factory methods?`

Are you tempted to repeat yourself because your reflective code needs to work for both constructors and factory methods in the same way?

Invokable offers an abstraction. The following code works with either Method or Constructor:
```java

invokable.isPublic();
invokable.getParameters();
invokable.invoke(object, args);
```

### `What's the return type of List.get(int) for List<String>?`

Invokable provides type resolution out of the box:
```java

Invokable<List<String>, ?> invokable = new TypeToken<List<String>>() {}.method(getMethod);
invokable.getReturnType(); // String.class
```

# Dynamic Proxies

### `newProxy()`

Utility method <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/Reflection.html#newProxy(java.lang.Class, java.lang.reflect.InvocationHandler)'><code>Reflection.newProxy(Class, InvocationHandler)</code></a> is a type safer and more convenient API to create Java dynamic proxies when only a single interface type is to be proxied.

JDK:
```java

Foo foo = (Foo) Proxy.newProxyInstance(
    Foo.class.getClassLoader(),
    new Class<?>[] {Foo.class},
    invocationHandler);
```

Guava:
```java

Foo foo = Reflection.newProxy(Foo.class, invocationHandler);
```

### `AbstractInvocationHandler`

Sometimes you may want your dynamic proxy to support equals(), hashCode() and toString() in the intuitive way, that is:
* A proxy instance is equal to another proxy instance if they are for the same interface types and have equal invocation handlers.
* A proxy's toString() delegates to the invocation handler's toString() for easier customization.

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/AbstractInvocationHandler.html'><code>AbstractInvocationHandler</code></a> implements this logic.

In addition, `AbstractInvocationHandler` ensures that the argument array passed to <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/AbstractInvocationHandler.html#handleInvocation(java.lang.Object, java.lang.reflect.Method, java.lang.Object[])'><code>handleInvocation(Object, Method, Object[])</code></a> is never null, thus less chance of `NullPointerException`.

# `ClassPath`

Strictly speaking, Java has no platform-independent way to browse through classes or class path resources. It is however sometimes desirable to be able to go through all classes under a certain package or project, for example, to check that certain project convention or constraint is being followed.

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/ClassPath.html'><code>ClassPath</code></a> is a utility that offers best-effort class path scanning. Usage is simple:

```java
ClassPath classpath = ClassPath.from(classloader); // scans the class path used by classloader
for (ClassPath.ClassInfo classInfo : classpath.getTopLevelClasses("com.mycomp.mypackage")) {
  ...
}
```

In the above example, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/ClassPath.ClassInfo.html'><code>ClassInfo</code></a> is a handle to the class to be loaded. It allows programmers to check the class name or package name and only load the class until necessary.

It's worth noting that `ClassPath` is a best-effort utility. It only scans classes in jar files or under a file system directory. Neither can it scan classes managed by custom class loaders that aren't URLClassLoader. So **don't use it for mission critical production tasks**.

# Class Loading
The utility method <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/reflect/Reflection.html#initialize(java.lang.Class...)'><code>Reflection.initialize(Class...)</code></a> ensures that the specified classes are initialized -- for example, any static initialization is performed.

The use of this method is a code smell, because static state hurts system maintainability and testability. In cases when you have no choice while inter-operating with a legacy framework, this method helps to keep the code less ugly.
