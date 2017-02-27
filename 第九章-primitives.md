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

# 字节转换方法
Guava 提供了若干方法，用来把原生类型按**大字节序**与字节数组相互转换。所有这些方法都是符号无关的，此外Booleans 没有提供任何下面的方法。

| 方法或字段签名                                | 描述                              |
| :-------------------------------------- | :--------------------------------------- |
| `int BYTES`                             | 常量：表示该原生类型需要的字节数 |
| `prim fromByteArray(byte[] bytes)`      | 使用字节数组的前 Prims.BYTES 个字节，按大字节序返回原生类型值；如果 `bytes.length <= Prims.BYTES`，抛出IllegalArgumentException  |
| `prim fromBytes(byte b1, ..., byte bk)` | 接受 `Prims.BYTES` 个字节参数，按大字节序返回原生类型值 |
| `byte[] toByteArray(prim value)`        | 按大字节序返回 `value `的字节数组 |

# 无符号支持
JDK 原生类型包装类提供了针对有符号类型的方法，而 `UnsignedInts` 和 `UnsignedLongs` 工具类提供了相应的无符号通用方法。`UnsignedInts` 和 `UnsignedLongs` 直接处理原生类型：使用时，由你自己保证只传入了无
符号类型的值。

此外，对 int 和 long，Guava 提供了无符号包装类([UnsignedInteger](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInteger.html) 和 [UnsignedLong](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLong.html)) ，来帮助你以极小的性能消耗，对有符号和无符号类型进行强制转换。

## 无符号通用工具方法
JDK 的原生类型包装类提供了有符号形式的类似方法。

| 方法签名 | 说明 |
|:----------|:------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String)'><code>int UnsignedInts.parseUnsignedInt(String)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String)</code></a> <table><thead><th> 按无符号十进制解析字符串 </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#parseUnsignedInt(java.lang.String, int)'><code>int UnsignedInts.parseUnsignedInt(String string, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#parseUnsignedLong(java.lang.String)'><code>long UnsignedLongs.parseUnsignedLong(String string, int radix)</code></a> </td><td> 按无符号的特定进制解析字符串 </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int)'><code>String UnsignedInts.toString(int)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long)'><code>String UnsignedLongs.toString(long)</code></a> </td><td> 数字按无符号十进制转为字符串 </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedInts.html#toString(int, int)'><code>String UnsignedInts.toString(int value, int radix)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/UnsignedLongs.html#toString(long, int)'><code>String UnsignedLongs.toString(long value, int radix)</code></a> </td><td> 数字按无符号特定进制转为字符串</td></tr></tbody></table>

<h2>无符号包装类</h2>
无符号包装类包含了若干方法，让使用和转换更容易。<br>
<br>
<table><thead><th> 方法签名 </th><th> 说明 </th></thead><tbody>
<tr><td> <code>UnsignedPrim plus(UnsignedPrim)</code>, <code>minus</code>, <code>times</code>, <code>dividedBy</code>, <code>mod</code> </td><td> 简单的算术运算。 </td></tr>
<tr><td> <code>UnsignedPrim valueOf(BigInteger)</code> </td><td> 按给定 <code>BigInteger</code> 返回无符号对象，若 <code>BigInteger</code>为负或不匹配，抛出 <code>IAE</code> </td></tr>
<tr><td> <code>UnsignedPrim valueOf(long)</code> </td><td> Returns the value from the <code>long</code> as an <code>UnsignedPrim</code>, or throw an <code>IAE</code> if the specified <code>long</code> is negative or does not fit. </td></tr>
<tr><td> <code>UnsignedPrim fromPrimBits(prim value)</code> </td><td> 将给定值视为无符号。 例如，UnsignedInteger.fromIntBits（1 << 31）的值为 2<sup>31</sup>，即使1 << 31为负数。</td></tr>
<tr><td> <code>BigInteger bigIntegerValue()</code> </td><td> 将此<code>UnsignedPrim</code>的值作为<code>BigInteger</code>获取。 </td></tr>
<tr><td> <code>toString()</code>, <code>toString(int radix)</code> </td><td> 返回此无符号值的字符串表示形式。 </td></tr>
