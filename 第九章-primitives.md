# Overview
Java 的原生类型就是指基本类型：

| `byte`  | `short`  | `int`  | `long`    |
| :------ | :------- | :----- | :-------- |
| `float` | `double` | `char` | `boolean` |

**在从 Guava 查找原生类型方法之前，可以先查查 [Arrays](http://docs.oracle.com/javase/1.5.0/docs/api/java/util/Arrays.html) 类，或者对应的基础类型包装类，如 [Integer](http://docs.oracle.com/javase/1.5.0/docs/api/java/lang/Integer.html)。**

原生类型不能当作对象或泛型的类型参数使用，这意味着许多通用方法都不能应用于它们。Guava 提供了一些通用工具，包括原生类型数组与集合 API 的交互，从原生类型和字节数组的相互转换，以及对某些原生类型的无符号形式的支持。

| 原生类型 | Guava 工具类 (都在`com.google.common.primitives` 包) |
| :------------- | :--------------------------------------- |
| `byte`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Bytes.html'><code>Bytes</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/SignedBytes.html '><code>SignedBytes</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedBytes.html'><code>UnsignedBytes</code></a> |
| `short`        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Shorts.html'><code>Shorts</code></a> |
| `int`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Ints.html'><code>Ints</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInteger.html'><code>UnsignedInteger</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html'><code>UnsignedInts</code></a> |
| `long`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Longs.html'><code>Longs</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLong.html'><code>UnsignedLong</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html'><code>UnsignedLongs</code></a> |
| `float`        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Floats.html'><code>Floats</code></a> |
| `double`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Doubles.html'><code>Doubles</code></a> |
| `char`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Chars.html'><code>Chars</code></a> |
| `boolean`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Booleans.html'><code>Booleans</code></a> |

Bytes 工具类没有定义任何区分有符号和无符号字节的方法，而是把它们都放到了 `SignedBytes` 和 `UnsignedBytes` 工具类中，因为字节类型的符号性比起其它类型要略微含糊一些。

`unsignedInts`和`UnsignedLongs`类中提供了`int`和`long`方法的无符号变量，但由于这些类型的大多数用法都是有符号的，Ints 和 Longs 类按照有符号形式处理方法的输入参数。

此外，Guava 为 `int` 和 `long` 的无符号形式提供了包装类，即 `UnsignedInteger` 和 `UnsignedLong`，以帮助你使用类型系统，以极小的性能消耗对有符号和无符号值进行强制转换。 这些类直接支持`BigInteger`风格的简单算术运算。

在本章下面描述的方法签名中，我们用 `Wrapper` 表示 JDK 包装类，`prim` 表示原生类型。（`Prims` 表示相应的Guava 工具类。）

# 原生类型数组工具
原生类型数组是处理原生类型集合的最有效方式（从内存和性能双方面考虑）。Guava 为此提供了许多工具方法。

| 方法签名                                | 描述                              | 类似方法                      | 可用性        |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :------------------ |
| `List<Wrapper> asList(prim... backingArray)` | 将原始数组封装为相应包装类型的`List`。 | [Arrays.asList](http://docs.oracle.com/javase/6/docs/api/java/util/Arrays.html#asList(T...)) | 符号无关`*` |
| `prim[] toArray(Collection<Wrapper> collection)` | 将集合复制到一个新的prim []中。 和`collection.toArray()`一样是线程安全的。 | [Collection.toArray()](http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#toArray()) | 符号无关    |
| `prim[] concat(prim[]... arrays)`        | 串联多个原生类型数组    | [Iterables.concat](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)) | 符号无关    |
| `boolean contains(prim[] array, prim target)` | 判断原生类型数组是否包含给定值 | [Collection.contains](http://docs.oracle.com/javase/6/docs/api/java/util/Collection.html#contains(java.lang.Object)) | 符号无关    |
| `int indexOf(prim[] array, prim target)` | 在`array`中找到值`target`的第一次出现的索引，如果没有这样的值，则返回-1。 | [List.indexOf](http://docs.oracle.com/javase/6/docs/api/java/util/List.html#indexOf(java.lang.Object)) | 符号无关    |
| `int lastIndexOf(prim[] array, prim target)` | 给定值在数组最后出现的索引，若不包含此值返回-1 | [List.lastIndexOf](http://docs.oracle.com/javase/6/docs/api/java/util/List.html#lastIndexOf(java.lang.Object)) | 符号无关    |
| `prim min(prim... array)`                | 数组中最小的值 | [Collections.min](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#min(java.util.Collection)) | 符号相关`**`  |
| `prim max(prim... array)`                | 数组中最大的值 | [Collections.max](http://docs.oracle.com/javase/6/docs/api/java/util/Collections.html#max(java.util.Collection)) | 符号相关      |
| `String join(String separator, prim... array)` | 构造一个包含`array`元素的字符串，用`separator`分隔。 | [Joiner.on(separator).join](https://github.com/google/guava/wiki/StringsExplained#joiner) | 符号相关     |
| `Comparator<prim[]> lexicographicalComparator()` | 按字典序比较原生类型数组的 Comparator  | [Ordering.natural().lexicographical()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Ordering.html#lexicographical()) | 符号相关      |

`*` 符号无关方法存在于： `Bytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`.  而 `UnsignedInts`, `UnsignedLongs`, `SignedBytes`, 或`UnsignedBytes`不存在。

`**` Sign-dependent methods are present in: `SignedBytes`, `UnsignedBytes`, `Shorts`, `Ints`, `Longs`, `Floats`, `Doubles`, `Chars`, `Booleans`, `UnsignedInts`, `UnsignedLongs`.  而 `Bytes`不存在。

# 通用工具方法
Guava提供了一些不属于JDK 6的基本实用程序。然而，这些方法中的一些在JDK 7中可用。

| 方法签名                        | 描述                              | 可用性                            |
| :------------------------------- | :--------------------------------------- | :--------------------------------------- |
| `int compare(prim a, prim b)`    | 传统的 `Comparator.compare` 方法，但针对原生类型。JDK7 的原生类型包装类也提供这样的方法 | 符号相关                           |
| `prim checkedCast(long value)`   | 把给定 long 值转为某一原生类型，若给定值不符合该原生类型，则抛出 `IllegalArgumentException` | 仅适用于符号相关的整型`*` |
| `prim saturatedCast(long value)` | 把给定 long 值转为某一原生类型，若给定值不符合则使用最接近的原生类型值 | 仅适用于符号相关的整型   |


`*`这里的整型包括  `byte`, `short`, `int`, `long`。不包括 `char`, `boolean`, `float`, 或 `double`.

注：`com.google.common.math.DoubleMath` 提供了舍入 `double` 的方法，支持多种舍入模式。请参阅[[文章| MathExplained＃floating-point-arithmetic]]。

# Byte conversion methods
Guava provides methods to convert primitive types to and from byte array representations **in big-endian order**.  All methods are sign-independent, except that `Booleans` provides none of these methods.

| Signature                               | Description                              |
| :-------------------------------------- | :--------------------------------------- |
| `int BYTES`                             | Constant representing the number of bytes needed to represent a `prim` value. |
| `prim fromByteArray(byte[] bytes)`      | Returns the `prim` value whose big-endian representation is the first `Prims.BYTES` bytes in the array `bytes`.  Throws an `IllegalArgumentException` if `bytes.length <= Prims.BYTES`. |
| `prim fromBytes(byte b1, ..., byte bk)` | Takes `Prims.BYTES` byte arguments.  Returns the `prim` value whose byte representation is the specified bytes in big-endian order. |
| `byte[] toByteArray(prim value)`        | Returns an array containing the big-endian byte representation of `value`. |

# Unsigned support
The `UnsignedInts` and `UnsignedLongs` utility classes provide some of the generic utilities that Java provides for signed types in their wrapper classes.  `UnsignedInts` and `UnsignedLongs` deal with the primitive type directly: it is up to you to make sure that only unsigned values are passed to these utilities.

Additionally, for `int` and `long`, Guava provides "unsigned" wrapper types ([UnsignedInteger](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInteger.html) and [UnsignedLong](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLong.html)) to help you enforce distinctions between unsigned and signed values in the type system, in exchange for a small performance penalty.

## Generic utilities
These methods' signed analogues are provided in the wrapper classes in the JDK.

| Signature | Explanation |
|:----------|:------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String)'><code>int UnsignedInts.parseUnsignedInt(String)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String)</code></a> <table><thead><th> Parses an unsigned value from a string in base 10. </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String, int)'><code>int UnsignedInts.parseUnsignedInt(String string, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String string, int radix)</code></a> </td><td> Parses an unsigned value from a string in the specified base. </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int)'><code>String UnsignedInts.toString(int)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long)'><code>String UnsignedLongs.toString(long)</code></a> </td><td> Returns a string representation of the unsigned value in base 10. </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int, int)'><code>String UnsignedInts.toString(int value, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long, int)'><code>String UnsignedLongs.toString(long value, int radix)</code></a> </td><td> Returns a string representation of the unsigned value in the specified base.</td></tr></tbody></table>

<h2>Wrapper</h2>
The provided unsigned wrapper types include a number of methods to make their use and conversion easier.<br>
<br>
<table><thead><th> Signature </th><th> Explanation </th></thead><tbody>
<tr><td> <code>UnsignedPrim plus(UnsignedPrim)</code>, <code>minus</code>, <code>times</code>, <code>dividedBy</code>, <code>mod</code> </td><td> Simple arithmetic operations. </td></tr>
<tr><td> <code>UnsignedPrim valueOf(BigInteger)</code> </td><td> Returns the value from a <code>BigInteger</code> as an <code>UnsignedPrim</code>, or throw an <code>IAE</code> if the specified <code>BigInteger</code> is negative or does not fit. </td></tr>
<tr><td> <code>UnsignedPrim valueOf(long)</code> </td><td> Returns the value from the <code>long</code> as an <code>UnsignedPrim</code>, or throw an <code>IAE</code> if the specified <code>long</code> is negative or does not fit. </td></tr>
<tr><td> <code>UnsignedPrim fromPrimBits(prim value)</code> </td><td> View the given value as unsigned.  For example, <code>UnsignedInteger.fromIntBits(1 &lt;&lt; 31)</code> has the value 2<sup>31</sup>, even though <code>1 &lt;&lt; 31</code> is negative as an <code>int</code>. </td></tr>
<tr><td> <code>BigInteger bigIntegerValue()</code> </td><td> Get the value of this <code>UnsignedPrim</code> as a <code>BigInteger</code>. </td></tr>
<tr><td> <code>toString()</code>, <code>toString(int radix)</code> </td><td> Returns a string representation of this unsigned value. </td></tr>
