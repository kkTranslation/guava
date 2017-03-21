# `TypeToken`

由于类型擦除，你不能够在运行时传递泛型类对象——你可能想强制转换它们，并假装这些对象是有泛型的，但
实际上它们没有。

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

### Introduction
Obtaining a `TypeToken` for a basic, raw class is as simple as

```java

TypeToken<String> stringTok = TypeToken.of(String.class);
TypeToken<Integer> intTok = TypeToken.of(Integer.class);
```

To obtain a `TypeToken` for a type with generics -- when you know the generic type arguments at compile time -- you use an empty anonymous inner class:

```java

TypeToken<List<String>> stringListTok = new TypeToken<List<String>>() {};
```

Or if you want to deliberately refer to a wildcard type:

```java

TypeToken<Map<?, ?>> wildMapTok = new TypeToken<Map<?, ?>>() {};
```

`TypeToken` provides a way to dynamically resolve generic type arguments, like this:

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

Note that if `mapToken` just returned `new TypeToken<Map<K, V>>()`, it could not actually reify the types assigned to `K` and `V`, so for example

```java

class Util {
  static <K, V> TypeToken<Map<K, V>> incorrectMapToken() {
    return new TypeToken<Map<K, V>>() {};
  }
}

System.out.println(Util.<String, BigInteger>incorrectMapToken());
// just prints out "java.util.Map<K, V>"
```

Alternately, you can capture a generic type with a (usually anonymous) subclass and resolve it against a context class that knows what the type parameters are.

```java

abstract class IKnowMyType<T> {
  TypeToken<T> type = new TypeToken<T>(getClass()) {};
}
...
new IKnowMyType<String>() {}.type; // returns a correct TypeToken<String>
```

With this technique, you can, for example, get classes that know their element types.

## Queries

`TypeToken` supports many of the queries supported by `Class`, but with generic constraints properly taken into account.

Supported query operations include:

| Method                    | Description                              |
| :------------------------ | :--------------------------------------- |
| `getType()`               | Returns the wrapped `java.lang.reflect.Type`. |
| `getRawType()`            | Returns the most-known runtime class.    |
| `getSubtype(Class<?>)`    | Returns some subtype of `this` that has the specified raw class.  For example, if this is `Iterable<String>` and the argument is `List.class`, the result will be `List<String>`. |
| ` getSupertype(Class<?>)` | Generifies the specified raw class to be a supertype of this type.  For example, if this is `Set<String>` and the argument is `Iterable.class`, the result will be `Iterable<String>`. |
| `isAssignableFrom(type)`  | Returns `true` if this type is assignable from the given type, taking into account generic parameters. `List<? extends Number>` is assignable from `List<Integer>`, but `List<String>` is not. |
| `getTypes()`              | Returns the set of all classes and interfaces that this type is or is a subtype of.  The returned `Set` also provides methods `classes()` and `interfaces()` to let you view only the superclasses and superinterfaces. |
| `isArray()`               | Checks if this type is known to be an array, such as `int[]` or even `<? extends A[]>`. |
| `getComponentType()`      | Returns the array component type.        |

### `resolveType`

`resolveType` is a powerful but complex query operation that can be used to "substitute" type arguments from the context token.  For example,
```java

TypeToken<Function<Integer, String>> funToken = new TypeToken<Function<Integer, String>>() {};

TypeToken<?> funResultToken = funToken.resolveType(Function.class.getTypeParameters()[1]));
  // returns a TypeToken<String>
```

`TypeToken` unifies the `TypeVariable`s provided by Java with the values of those type variables from the "context" token.  This can be used to generically deduce the return types of methods on a type:

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
