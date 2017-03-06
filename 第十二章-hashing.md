# 概述

Java 内建的散列码[hash code]概念被限制为 32 位，并且没有分离散列算法和它们所作用的数据，因此很难用备选算法进行替换。此外，使用 Java 内建方法实现的散列码通常是劣质的，部分是因为它们最终都依赖于 JDK类中已有的劣质散列码。

`Object.hashCode` 往往很快，但是在预防碰撞上却很弱，也没有对分散性的预期。这使得它们很适合在散列表中运用，因为额外碰撞只会带来轻微的性能损失，同时差劲的分散性也可以容易地通过再散列来纠正（Java 中所有合理的散列表都用了再散列方法）。然而，在简单散列表以外的散列运用中，`Object.hashCode` 几乎总是达不到要求——因此，有了[com.google.common.hash](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/package-summary.html)包。

# 散列包的组成
在这个包的 Java doc 中，我们可以看到很多不同的类，但是文档中没有明显地表明它们是怎样 一起配合工作
的。在介绍散列包中的类之前，让我们先来看下面这段代码范例：
```java

HashFunction hf = Hashing.md5();
HashCode hc = hf.newHasher()
       .putLong(id)
       .putString(name, Charsets.UTF_8)
       .putObject(person, personFunnel)
       .hash();
```

### HashFunction
[HashFunction](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashFunction.html) 是一个单纯的（引用透明的）、无状态的方法，它把任意的数据块映射到固定数目的位指，并且保证相同的输入一定产生相同的输出，不同的输入尽可能产生不同的输出。
### Hasher
`HashFunction` 的实例可以提供有状态的 [Hasher](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hasher.html)，`Hasher` 提供了流畅的语法把数据添加到散列运算，然后获取散列值。`Hasher` 可以接受所有原生类型、字节数组、字节数组的片段、字符序列、特定字符集的字符序列等等，或者任何给定了 `Funnel` 实现的对象。

`Hasher` 实现了 `PrimitiveSink` 接口，这个接口为接受原生类型流的对象定义了 `fluent` 风格的 API

### Funnel
[Funnel](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Funnel.html) 描述了如何把一个具体的对象类型分解为原生字段值，从而写入 `PrimitiveSink`。比如，如果我们有这样一个类：
```java

class Person {
  final int id;
  final String firstName;
  final String lastName;
  final int birthYear;
}
```
它对应的 `Funnel` 实现可能是：
```java
Funnel<Person> personFunnel = new Funnel<Person>() {
  @Override
  public void funnel(Person person, PrimitiveSink into) {
    into
      .putInt(person.id)
      .putString(person.firstName, Charsets.UTF_8)
      .putString(person.lastName, Charsets.UTF_8)
      .putInt(birthYear);
  }
};
```

注：`putString(“abc”, Charsets.UTF_8).putString(“def”, Charsets.UTF_8)`完全等同于 `putString(“ab”, Charsets.UTF_8).putString(“cdef”, Charsets.UTF_8)`，因为它们提供了相同的字节序列。这可能带来预料之外的散列冲突。增加某种形式的分隔符有助于消除散列冲突。
### HashCode
一旦 `Hasher` 被赋予了所有输入，就可以通过 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hasher.html#hash()'><code>hash()</code></a> 方法获取 [HashCode](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html)实例（多次调用 `hash()`方法的结果是不确定的）。`HashCode` 可以通过<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asInt()'><code>asInt()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asLong()'><code>asLong()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asBytes()'><code>asBytes()</code></a> 方法来做相等性检测，此外， <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#writeBytesTo(byte[], int, int)'><code>writeBytesTo(array, offset, maxLength)</code></a>把散列值的前 `maxLength`字节写入字节数组。

## 布鲁姆过滤器 BloomFilter
布鲁姆过滤器是哈希运算的一项优雅运用，它可以简单地基于 `Object.hashCode()`实现。简而言之，布鲁姆过滤
器是一种概率数据结构，它允许你检测某个对象是一定不在过滤器中，还是可能已经添加到过滤器了。 [布鲁姆过滤器的维基页面](http://en.wikipedia.org/wiki/Bloom_filter) 对此作了全面的介绍，同时我们推荐 github 中的一个 [教程](http://llimllib.github.com/bloomfilter-tutorial/).

Guava 散列包有一个内建的布鲁姆过滤器实现，你只要提供 Funnel 就可以使用它。你可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#create(com.google.common.hash.Funnel, int, double)'><code>create(Funnel funnel, int expectedInsertions, double falsePositiveProbability)</code></a>方法获取[BloomFilter&lt;T&gt;](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html), 或者只接受默认的3％的概率。`BloomFilter<T>` 提供了 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#mightContain(T)'><code>boolean mightContain(T)</code></a> 和 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#put(T)'><code>void put(T)</code></a>, 它们的含义都不言自明了。

```java
BloomFilter<Person> friends = BloomFilter.create(personFunnel, 500, 0.01);
for(Person friend : friendsList) {
  friends.put(friend);
}
// much later
if (friends.mightContain(dude)) {
  //dude不是friend还运行到这里的概率为1%
//在这儿，我们可以在做进一步精确检查的同时触发一些异步加载
}
```

# `Hashing`
`Hashing` 类提供了若干散列函数，以及运算 `HashCode` 对象的工具方法。

## 已提供的散列函数
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#md5()'><code>md5()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#murmur3_128()'><code>murmur3_128()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#murmur3_32()'><code>murmur3_32()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha1()'><code>sha1()</code></a> |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha256()'><code>sha256()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha512()'><code>sha512()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#goodFastHash(int)'><code>goodFastHash(int bits)</code></a> |                                          |

## `HashCode`  运算
| 方法                                   | 描述                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#combineOrdered(java.lang.Iterable)'><code>HashCode combineOrdered(Iterable&lt;HashCode&gt;)</code></a> | 以有序方式联接散列码，如果两个散列集合用该方法联接出的散列码相
同，那么散列集合的元素可能是顺序相等的 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#combineUnordered(java.lang.Iterable)'><code>HashCode combineUnordered(Iterable&lt;HashCode&gt;)</code></a> | 以无序方式联接散列码，如果两个散列集合用该方法联接出的散列码相
同，那么散列集合的元素可能在某种排序下是相等的 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#consistentHash(com.google.common.hash.HashCode, int)'><code>int consistentHash(HashCode, int buckets)</code></a> | 为给定的”桶”大小返回一致性哈希值。当”桶”增长时，该方法保证
最小程度的一致性哈希值变化。详见 <a href='http://en.wikipedia.org/wiki/Consistent_hashing'>Wikipedia</a>  |
