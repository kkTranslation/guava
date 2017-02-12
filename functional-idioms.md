# 说明[Caveats]
截至 JDK7，Java 中也只能通过笨拙冗长的匿名类来达到近似函数式编程的效果。预计 JDK8 中会有所改变，但 Guava 现在就想给 JDK5 以上用户提供这类支持。过度使用 Guava 函数式编程会导致冗长、混乱、可读性差而且低效的代码。这是迄今为止最容易（也是最经常）被滥用的部分，如果你想通过函数式风格达成一行代码，致使这行代码长到荒唐，Guava 团队会泪流满面。

比较如下代码：
```java

Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
Predicate<String> allCaps = new Predicate<String>() {
  public boolean apply(String string) {
    return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
  }
};
Multiset<Integer> lengths = HashMultiset.create(
  Iterables.transform(Iterables.filter(strings, allCaps), lengthFunction));
```
或 `FluentIterable` 的版本
```java

Multiset<Integer> lengths = HashMultiset.create(
  FluentIterable.from(strings)
    .filter(new Predicate<String>() {
       public boolean apply(String string) {
         return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
       }
     })
    .transform(new Function<String, Integer>() {
       public Integer apply(String string) {
         return string.length();
       }
     }));
```
with:
```java
Multiset<Integer> lengths = HashMultiset.create();
for (String string : strings) {
  if (CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string)) {
    lengths.add(string.length());
  }
}
```

即使用了静态导入，甚至把 Function 和 Predicate 的声明放到别的文件，第一种代码实现仍然不简洁，可读性差并且效率较低。

截至 JDK7，命令式代码仍应是默认和第一选择。不应该随便使用函数式风格，除非你绝对确定以下两点之一：

* 使用函数式风格以后，整个工程的代码行会净减少。在上面的例子中，函数式版本用了 11 行， 命令式代码用了 6 行，把函数的定义放到另一个文件或常量中，并不能帮助减少总代码行。
* 为了提高效率，转换集合的结果需要懒视图，而不是明确计算过的集合。此外，确保你已经阅读和重读了 Eff
  ective Java 的第 55 条，并且除了阅读本章后面的说明，你还真正做了性能测试并且有测试数据来证明函数式版本更快。

请务必确保，当使用 Guava 函数式的时候，用传统的命令式做同样的事情不会更具可读性。尝试把代码写下来，看看它是不是真的那么糟糕？会不会比你想尝试的极其笨拙的函数式 更具可读性。

# Functions and Predicates
This article discusses only those Guava features dealing directly with `Function` and `Predicate`.  Some other utilities are associated with the "functional style," such as concatenation and other methods which return views in constant time.  Try looking in the [[collection utilities|CollectionUtilitiesExplained]] article.

Guava 提供两个基本的函数式接口：
* `Function<A, B>`, 它声明了单个方法`B apply(A input)`.  `Function` 对象通常被预期为引用透明的——没有副作用——并且引用透明性中的”相等”语义与 equals 一致，如  `a.equals(b)` 意味着  `function.apply(a).equals(function.apply(b))`.
* `Predicate<T>`, 它声明了单个方法 `boolean apply(T input)`.  `Predicate` 对象通常也被预期为无副作用函数，并且”相等”语义与 equals 一致。

### 特殊的断言
字符类型有自己特定版本的 `Predicate`—— <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html'><code>CharMatcher</code></a>，它通常更高效，并且在某些需求方面更有用。CharMatcher 实现了 `Predicate`，可以当作 `Predicate`一样使用，要把 `Predicate`转成 CharMatcher，可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#forPredicate(com.google.common.base.Predicate)'><code>CharMatcher.forPredicate</code></a>。更多细节请参考第 6 章-字符串处理。此外，对可比较类型和基于比较逻辑的 `Predicate`，Range 类可以满足大多数需求——它表示一个不可变区间。Range 类实现了 `Predicate`，用以判断值是否在区间内。例如，Range.atMost(2)就是个完全合法的 `Predicate`。更多使用 Range 的细节请参照第8章。

Additionally, for comparable types and comparison-based predicates, most needs can be fulfilled using the `Range` type, which implements an immutable interval.  The `Range` type implements `Predicate`, testing containment in the range.  For example, `Ranges.atMost(2)` is a perfectly valid `Predicate<Integer>`.  More details on using ranges can be found [[in the corresponding article|RangesExplained]].

### 操作 Functions 和 Predicates

`Function` 提供简便的 [Functions](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html)构造和操作方法，包括：

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#forMap(java.util.Map)'><code>forMap(Map&lt;A, B&gt;)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#compose(com.google.common.base.Function, com.google.common.base.Function)'><code>compose(Function&lt;B, C&gt;, Function&lt;A, B&gt;)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#constant(E)'><code>constant(T)</code></a> |  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#identity()'><code>identity()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Functions.html#toStringFunction()'><code>toStringFunction()</code></a> |
|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|



相应地，Predicates 提供了更多构造和处理 Predicate 的方法，下面是一些例子：

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#instanceOf(java.lang.Class)'><code>instanceOf(Class)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#assignableFrom(java.lang.Class)'><code>assignableFrom(Class)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#contains(java.util.regex.Pattern)'><code>contains(Pattern)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#in(java.util.Collection)'><code>in(Collection)</code></a> |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#isNull()'><code>isNull()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#alwaysFalse()'><code>alwaysFalse()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#alwaysTrue()'><code>alwaysTrue()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#equalTo(T)'><code>equalTo(Object)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#compose(com.google.common.base.Predicate, com.google.common.base.Function)'><code>compose(Predicate, Function)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#and(com.google.common.base.Predicate...)'><code>and(Predicate...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#or(com.google.common.base.Predicate...)'><code>or(Predicate...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Predicates.html#not(com.google.common.base.Predicate)'><code>not(Predicate)</code></a> |

细节请参考 Javadoc。

# 使用函数式编程
Guava 提供了很多工具方法，以便用 Function 或 Predicate 操作集合。这些方法通常可以在集合工具类找到 。如`Iterables`, `Lists`, `Sets`, `Maps`, `Multimaps`.

## Predicates
断言的最基本应用就是过滤集合。所有 Guava 过滤方法都返回”视图”——译者注：即并非用一个新的集合表示过滤，而只是基于原集合的视图。

| Collection type | Filter method |
| --------------- | ------------- |
|                 |               |

| :---------------- | :-------------- |
| ----------------- | --------------- |
|                   |                 |

| `Iterable`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#filter(java.lang.Iterable, com.google.common.base.Predicate)'><code>Iterables.filter(Iterable, Predicate)</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#filter(com.google.common.base.Predicate)'><code>FluentIterable.filter(Predicate)</code></a> 