TODO: lots more examples

# 示例
```java

List<Double> scores;
Iterable<Double> belowMedianScores = Iterables.filter(scores, Range.lessThan(median));
...
Range<Integer> validGrades = Range.closed(1, 12);
for(int grade : ContiguousSet.create(validGrades, DiscreteDomain.integers())) {
  ...
}
```

# 简介
区间，有时也称为范围，是特定域中的凸性（非正式说法为连续的或不中断的）部分。在形式上，凸性表示对
`a<=b<=c`, `range.contains(a) && range.contains(c)`意味着 `range.contains(b)`。

区间可以延伸至无限——例如，范围”x>3″包括任意大于3的值——也可以被限制为有限，如” 2<=x<5″。Guava 用更紧凑的方法表示范围，有数学背景的程序员对此是耳熟能详的：

* (a..b) = {x | a < x < b}
* [a..b] = {x | a <= x <= b}
    * [a..b) = {x | a <= x < b}
    * (a..b] = {x | a < x <= b}
    * (a..+∞) = {x | x > a}
    * [a..+∞) = {x | x >= a}
    * (-∞..b) = {x | x < b}
    * (-∞..b] = {x | x <= b}
    * (-∞..+∞) = all values

上面的 a、b 称为端点 。为了提高一致性，Guava 中的 `Range` 要求上端点不能小于下端点。只有当边界中的至少一个被关闭时，端点才可以是相等的：

* [a..a] : 单元素区间
* [a..a); (a..a] : 空区间，但它们是有效的
    * (a..a) : 无效区间

Guava 用类型[Range&lt;C&gt;](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html)表示区间。所有区间实现都是不可变类型。

# 构建区间
区间实例可以由 [Range](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html)类的静态方法获取： 

| (a..b)     | [open(C, C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#open(java.lang.Comparable,java.lang.Comparable)) |
| :--------- | :--------------------------------------- |
| `[`a..b`]` | [closed(C, C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#closed(java.lang.Comparable,java.lang.Comparable)) |
| `[`a..b)   | [closedOpen(C, C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#closedOpen(java.lang.Comparable,java.lang.Comparable)) |
| (a..b`]`   | [openClosed(C, C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#openClosed(java.lang.Comparable,java.lang.Comparable)) |
| (a..+∞)    | [greaterThan(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#greaterThan(C)) |
| `[`a..+∞)  | [atLeast(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#atLeast(C)) |
| (-∞..b)    | [lessThan(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#lessThan(C)) |
| (-∞..b`]`  | [atMost(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#atMost(C)) |
| (-∞..+∞)   | [all()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#all()) |

```java

Range.closed("left", "right"); // 字典序在"left"和"right"之间的字符串，闭区间
Range.lessThan(4.0); // 严格小于4.0的double值
```

另外，Range实例可以通过显式传递绑定类型来构造：

| 有界区间                     | [range(C, BoundType, C, BoundType)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#range(java.lang.Comparable,com.google.common.collect.BoundType,java.lang.Comparable,com.google.common.collect.BoundType)) |
| :--------------------------------------- | :--------------------------------------- |
| 无上界区间： ((a..+∞) 或 `[`a..+∞))  | [downTo(C, BoundType)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#downTo(java.lang.Comparable,com.google.common.collect.BoundType)) |
| 无下界区间： ((-∞..b) or (-∞..b]) | [upTo(C, BoundType)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#upTo(java.lang.Comparable,com.google.common.collect.BoundType)) |

这里的[BoundType](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BoundType.html) 是一个枚举类型，包含 `CLOSED` 和`OPEN`两个值。

```java

Range.downTo(4, boundType); // (a..+∞)或[a..+∞)，取决于boundType
Range.range(1, CLOSED, 4, OPEN); // [1..4)，等同于Range.closedOpen(1, 4)
```

# Operations
The fundamental operation of a `Range` is its [contains(C)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#contains(C)) methods, which behaves exactly as you might expect.  Additionally, a `Range` may be used as a `Predicate`, and used in [[functional idioms|FunctionalExplained]].  Any `Range` also supports `containsAll(Iterable<? extends C>)`.

```java

Range.closed(1, 3).contains(2); // returns true
Range.closed(1, 3).contains(4); // returns false
Range.lessThan(5).contains(5); // returns false
Range.closed(1, 4).containsAll(Ints.asList(1, 2, 3)); // returns true
```

## Query Operations

To look at the endpoints of a range, `Range` exposes the following methods:
* [hasLowerBound()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#hasLowerBound()) and [hasUpperBound()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#hasUpperBound()), which check if the range has the specified endpoints, or goes on "through infinity."
* [lowerBoundType()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#lowerBoundType()) and [upperBoundType()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#upperBoundType()) return the `BoundType` for the corresponding endpoint, which can be either `CLOSED` or `OPEN`.  If this range does not have the specified endpoint, the method throws an `IllegalStateException`.
    * [lowerEndpoint()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#lowerEndpoint()) and [upperEndpoint()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#upperEndpoint()) return the endpoints on the specified end, or throw an `IllegalStateException` if the range does not have the specified endpoint.
    * [isEmpty()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#isEmpty()) tests if the range is empty, that is, it has the form `[`a,a) or (a,a`]`.

```java

Range.closedOpen(4, 4).isEmpty(); // returns true
Range.openClosed(4, 4).isEmpty(); // returns true
Range.closed(4, 4).isEmpty(); // returns false
Range.open(4, 4).isEmpty(); // Range.open throws IllegalArgumentException

Range.closed(3, 10).lowerEndpoint(); // returns 3
Range.open(3, 10).lowerEndpoint(); // returns 3
Range.closed(3, 10).lowerBoundType(); // returns CLOSED
Range.open(3, 10).upperBoundType(); // returns OPEN
```

## Interval Operations
### `encloses`
The most basic relation on ranges is <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#encloses(com.google.common.collect.Range)'><code>encloses(Range)</code></a>, which is true if the bounds of the inner range do not extend outside the bounds of the outer range.  This is solely dependent on comparisons between the endpoints!

* [3..6] encloses [4..5]
* (3..6) encloses (3..6)
    * [3..6] encloses [4..4) (even though the latter is empty)
    * (3..6] does not enclose [3..6]
    * [4..5] does not enclose (3..6) **even though it contains every value contained by the latter range**, although use of discrete domains can address this (see below)
    * [3..6] does not enclose (1..1] **even though it contains every value contained by the latter range**

`encloses` is a [[partial ordering|GuavaTermsExplained#partial-ordering]].

Given this, `Range` provides the following operations:

### `isConnected`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#isConnected(com.google.common.collect.Range)'><code>Range.isConnected(Range)</code></a>, which tests if these ranges are _connected_.  Specifically, `isConnected` tests if there is some range enclosed by both of these ranges, but this is equivalent to the mathematical definition that the union of the ranges must form a connected set (except in the special case of empty ranges).

`isConnected` is a [[reflexive|GuavaTermsExplained#reflexive]], [[symmetric|GuavaTermsExplained#symmetric]] [[relation|GuavaTermsExplained#relation]].

```java

Range.closed(3, 5).isConnected(Range.open(5, 10)); // returns true
Range.closed(0, 9).isConnected(Range.closed(3, 4)); // returns true
Range.closed(0, 5).isConnected(Range.closed(3, 9)); // returns true
Range.open(3, 5).isConnected(Range.open(5, 10)); // returns false
Range.closed(1, 5).isConnected(Range.closed(6, 10)); // returns false
```

### `intersection`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#intersection(com.google.common.collect.Range)'><code>Range.intersection(Range)</code></a> returns the maximal range enclosed by both this range and other (which exists iff these ranges are connected), or if no such range exists, throws an `IllegalArgumentException`.

`intersection` is a [[commutative|GuavaTermsExplained#commutative]], [[associative|GuavaTermsExplained#associative]] [[operation|GuavaTermsExplained#binary-operation]].

```java

Range.closed(3, 5).intersection(Range.open(5, 10)); // returns (5, 5]
Range.closed(0, 9).intersection(Range.closed(3, 4)); // returns [3, 4]
Range.closed(0, 5).intersection(Range.closed(3, 9)); // returns [3, 5]
Range.open(3, 5).intersection(Range.open(5, 10)); // throws IAE
Range.closed(1, 5).intersection(Range.closed(6, 10)); // throws IAE
```

### `span`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#span(com.google.common.collect.Range)'><code>Range.span(Range)</code></a> returns the minimal range that encloses both this range and other.  If the ranges are both connected, this is their union.

`span` is a [[commutative|GuavaTermsExplained#commutative]], [[associative|GuavaTermsExplained#associative]], and [[closed|GuavaTermsExplained#closed]] [[operation|GuavaTermsExplained#binary-operation]].

```java

Range.closed(3, 5).span(Range.open(5, 10)); // returns [3, 10)
Range.closed(0, 9).span(Range.closed(3, 4)); // returns [0, 9]
Range.closed(0, 5).span(Range.closed(3, 9)); // returns [0, 9]
Range.open(3, 5).span(Range.open(5, 10)); // returns (3, 10)
Range.closed(1, 5).span(Range.closed(6, 10)); // returns [1, 10]
```

# Discrete Domains
Some types, but not all Comparable types, are _discrete_, meaning that ranges bounded on both sides can be enumerated.

In Guava, a [DiscreteDomain&lt;C&gt;](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/DiscreteDomain.html) implements discrete operations for type `C`.  A discrete domain always represents the entire set of values of its type; it cannot represent partial domains such as "prime integers", "strings of length 5," or "timestamps at midnight."

The [DiscreteDomain](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/DiscreteDomain.html) class provides `DiscreteDomain` instances:

| Type      | DiscreteDomain                           |
| :-------- | :--------------------------------------- |
| `Integer` | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/DiscreteDomain.html#integers()'><code>integers()</code></a> |
| `Long`    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/DiscreteDomain.html#longs()'><code>longs()</code></a> |

Once you have a `DiscreteDomain`, you can use the following `Range` operations:
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ContiguousSet.html#create(com.google.common.collect.Range, com.google.common.collect.DiscreteDomain)'><code>ContiguousSet.create(range, domain)</code></a>: view a `Range<C>` as an `ImmutableSortedSet<C>`, with a few extra operations thrown in.  (Does not work for unbounded ranges, unless the type itself is bounded.)
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#canonical(com.google.common.collect.DiscreteDomain)'><code>canonical(domain)</code></a>: put ranges in a "canonical form."  If `ContiguousSet.create(a, domain).equals(ContiguousSet.create(b, domain))` and `!a.isEmpty()`, then `a.canonical(domain).equals(b.canonical(domain))`.  (This does _not_, however, imply `a.equals(b)`.)

```java
ImmutableSortedSet<Integer> set = ContiguousSet.create(Range.open(1, 5), DiscreteDomain.integers());
// set contains [2, 3, 4]

ContiguousSet.create(Range.greaterThan(0), DiscreteDomain.integers());
// set contains [1, 2, ..., Integer.MAX_VALUE]
```
Note that `ContiguousSet.create` does not _actually_ construct the entire range, but instead returns a view of the range as a set.

## Your Own DiscreteDomains

You can make your own `DiscreteDomain` objects, but there are several important aspects of the `DiscreteDomain` contract that you _must_ remember.

* A discrete domain always represents the entire set of values of its type; it cannot represent partial domains such as "prime integers" or "strings of length 5."  So you cannot, for example, construct a `DiscreteDomain` to view a set of days in a range, with a JODA `DateTime` that includes times up to the second: because this would not contain all elements of the type.
* A `DiscreteDomain` may be infinite -- a `BigInteger` `DiscreteDomain`, for example.  In this case, you should use the default implementation of `minValue()` and `maxValue()`, which throw a `NoSuchElementException`.  This forbids you from using the `ContiguousSet.create` method on an infinite range, however!

# What if I need a `Comparator`?
We wanted to strike a very specific balance in `Range` between power and API complexity, and part of that involved not providing a `Comparator`-based interface: we don't need to worry about how ranges based on different comparators interact; the API signatures are all significantly simplified; things are just nicer.

On the other hand, if you think you want an arbitrary `Comparator`, you can do one of the following:

* Use a general `Predicate` and not `Range`.  (Since `Range` implements the `Predicate` interface, you can use `Predicates.compose(range, function)` to get a `Predicate`.)
* Use a wrapper class around your objects that defines the desired ordering.
