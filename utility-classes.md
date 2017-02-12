TODO: Queues, Tables

任何具有JDK集合框架经验的程序员都知道并喜欢 <a href='http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html'><code>java.util.Collections</code></a> 中提供的工具方法。 Guava提供了更多的工具方法：适用于所有集合的静态方法。这是 Guava 最流行和成熟的部分之一。

我们用相对直观的方式把工具类与特定集合接口的对应关系归纳如下：

| 集合接口    | JDK 还是 Guava?                          | 对应的Guava工具类        |
| :----------- | :------------------------------------- | :--------------------------------------- |
| `Collection` | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Collections2.html'><code>Collections2</code></a> (不要和 `java.util.Collections`混淆) |
| `List`       | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html'><code>Lists</code></a> |
| `Set`        | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> |
| `SortedSet`  | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> |
| `Map`        | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> |
| `SortedMap`  | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> |
| `Queue`      | JDK                                    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Queues.html'><code>Queues</code></a> |
| [[Multiset   | NewCollectionTypesExplained#multiset]] | Guava                                    |
| [[Multimap   | NewCollectionTypesExplained#multimap]] | Guava                                    |
| [[BiMap      | NewCollectionTypesExplained#bimap]]    | Guava                                    |
| [[Table      | NewCollectionTypesExplained#table]]    | Guava                                    |

**_在找类似转化、过滤的方法？请看第四章，函数式风格。_**

# 静态工厂方法
在 JDK 7之前，构造新的范型集合时要讨厌地重复声明范型：
```java

List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();
```
我想我们都认为这很讨厌。因此 Guava 提供了能够推断范型的静态工厂方法：
```java

List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();
```

可以肯定的是，JDK7 版本的钻石操作符(<>)没有这样的麻烦：

```java

List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();
```

但 Guava 的静态工厂方法远不止这么简单。使用工厂方法模式，我们可以非常方便地使用起始元素初始化集合。

```java

Set<Type> copySet = Sets.newHashSet(elements);
List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");
```

此外，通过为工厂方法命名(Effective Java 第一条)，我们可以提高初始化集合到大小的可读性：

```java

List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
```

下面列出了提供的精确静态工厂方法及其相应的实用程序类。

注意：Guava 引入的新集合类型没有暴露原始构造器，也没有在工具类中提供初始化方法。而是直接在集合类中提供了静态工厂方法，例如：

```java

Multiset<String> multiset = HashMultiset.create();
```

# Iterables
在可能的情况下，Guava 提供的工具方法更偏向于接受 `Iterable` 而不是 `Collection` 类型。在 Google，对于不存放在主存的集合——比如从数据库或其他数据中心收集的结果集，因为实际上还没有攫取全部数据，这类结果集都不能支持类似 `size()`的操作 ——通常都不会用 Collection 类型来表示。

因此，您可能期望看到的所有集合支持的许多操作都可以在 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html'><code>Iterables</code></a>.  中找到。 此外，大多数 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterators.html'><code>Iterators</code></a> 方法在接受原始迭代器的迭代器中有相应的版本。

Iterables类中绝大多数操作都是惰性的：它们只在绝对必要时才推进支持迭代。 自己返回Iterables的方法返回延迟计算的视图，而不是在内存中显式构造集合。

从Guava 1.2开始，Iterables由 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/FluentIterable.html'><code>FluentIterable</code></a> 类补充，该类包装了一个Iterable，并为许多这些操作提供了一个“fluent”（链式调用）语法。

下面列出了一些最常用的工具方法，但更多 Iterables 的函数式方法将在第四章讨论。

### 常规方法
| Method                                   | Description                              | See Also                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable)'><code>concat(Iterable&lt;Iterable&gt;)</code></a> | 串联多个 iterables 的懒视图* | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#concat(java.lang.Iterable...)'><code>concat(Iterable...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#frequency(java.lang.Iterable, java.lang.Object)'><code>frequency(Iterable, Object)</code></a> | 返回对象在 iterable 中出现的次数 | 与 Collections.frequency (Collecti
on, Object)比较； |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#partition(java.lang.Iterable, int)'><code>partition(Iterable, int)</code></a> | 把 iterable 按指定大小分割，得到的子集都不能进行修改操作 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#partition(java.util.List, int)'><code>Lists.partition(List, int)</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#paddedPartition(java.lang.Iterable, int)'><code>paddedPartition(Iterable, int)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getFirst(java.lang.Iterable, T)'><code>getFirst(Iterable, T default)</code></a> | 返回 iterable 的第一个元素，若 iterable 为空则返回默认值| 与I`terable.iterator(). next()`比较;<br><a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#first()'><code>FluentIterable.first()</code></a> <br>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable)'><code>getLast(Iterable)</code></a> </td><td> 返回 iterable 的最后一个元素，若 iterable 为空则抛出NoSuchElementException </td><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getLast(java.lang.Iterable, T)'><code>getLast(Iterable, T default)</code></a><br><a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#last()'><code>FluentIterable.last()</code></a> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#elementsEqual(java.lang.Iterable, java.lang.Iterable)'><code>elementsEqual(Iterable, Iterable)</code></a> </td><td> 如果两个 iterable 中的所有元素相等
且顺序一致，返回 true </td><td> 与List.equals(Object)比较</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#unmodifiableIterable(java.lang.Iterable)'><code>unmodifiableIterable(Iterable)</code></a> </td><td> 返回 iterable 的不可变视图 </td><td>  与<code> Collections. unmodifiableColle
ction(Collection)</code>比较 </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#limit(java.lang.Iterable, int)'><code>limit(Iterable, int)</code></a> </td><td> Returns an <code>Iterable</code> 限制 iterable 的元素个数限制给定值 </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#limit(int)'><code>FluentIterable.limit(int)</code></a> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable)'><code>getOnlyElement(Iterable)</code></a> </td><td> 获取 <code>Iterable</code> 中唯一的元素，如果 iter
able 为空或有多个元素，则快速失败</td><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#getOnlyElement(java.lang.Iterable, T)'><code>getOnlyElement(Iterable, T default)</code></a> </td></tr></tbody></table>

```java

Iterable<Integer> concatenated = Iterables.concat(
  Ints.asList(1, 2, 3),
  Ints.asList(4, 5, 6));
// concatenated 包括元素 1, 2, 3, 4, 5, 6

String lastAdded = Iterables.getLast(myLinkedHashSet);

String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
  // 如果set不是单元素集，就会出错了！
```

### Collection-Like
通常来说，Collection 的实现天然支持操作其他 Collection，但却不能操作 Iterable。

下面的方法中，如果传入的 Iterable 是一个 `Collection`实例，则实际操作将会委托给相应的 `Collection` 接口方
法。例如，往 `Iterables.size` 方法传入是一个 `Collection` 实例，它不会真的遍历 iterator 获取大小，而是直接调
用 `Collection.size`。

| 方法                                   | 类似的 Collection 方法      | 等价的 FluentIterable 方法              |
| :--------------------------------------- | :--------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#addAll(java.util.Collection, java.lang.Iterable)'><code>addAll(Collection addTo, Iterable toAdd)</code></a> | `Collection.addAll(Collection)`    |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#contains(java.lang.Iterable, java.lang.Object)'><code>contains(Iterable, Object)</code></a> | `Collection.contains(Object)`      | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#contains(java.lang.Object)'><code>FluentIterable.contains(Object)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#removeAll(java.lang.Iterable, java.util.Collection)'><code>removeAll(Iterable removeFrom, Collection toRemove)</code></a> | `Collection.removeAll(Collection)` |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#retainAll(java.lang.Iterable, java.util.Collection)'><code>retainAll(Iterable removeFrom, Collection toRetain)</code></a> | `Collection.retainAll(Collection)` |                                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#size(java.lang.Iterable)'><code>size(Iterable)</code></a> | `Collection.size()`                | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#size()'><code>FluentIterable.size()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#toArray(java.lang.Iterable, java.lang.Class)'><code>toArray(Iterable, Class)</code></a> | `Collection.toArray(T[])`          | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toArray(java.lang.Class)'><code>FluentIterable.toArray(Class)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#isEmpty(java.lang.Iterable)'><code>isEmpty(Iterable)</code></a> | `Collection.isEmpty()`             | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#isEmpty()'><code>FluentIterable.isEmpty()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#get(java.lang.Iterable, int)'><code>get(Iterable, int)</code></a> | `List.get(int)`                    | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#get(int)'><code>FluentIterable.get(int)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterables.html#toString(java.lang.Iterable)'><code>toString(Iterable)</code></a> | `Collection.toString()`            | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toString()'><code>FluentIterable.toString()</code></a> |

### `FluentIterable`
除了上面和第四章提到的方法，`FluentIterable`有一些方便的方法复制到不可变集合：

| `ImmutableList` | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableList()>`toImmutableList()`</a'> <br>
<tr><td> <code>ImmutableSet</code> </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableSet()'><code>toImmutableSet()</code></a> </td></tr>
<tr><td> <code>ImmutableSortedSet</code> </td><td> <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/FluentIterable.html#toImmutableSortedSet(java.util.Comparator)'><code>toImmutableSortedSet(Comparator)</code></a> </td></tr></tbody></table>

### Lists
除了静态工厂方法和函数式编程方法外， <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html'><code>Lists</code></a> 还为List对象提供了许多有价值的实用方法。

| 方法                                   | 描述                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#partition(java.util.List, int)'><code>partition(List, int)</code></a> | 返回基础列表的视图，分区为指定大小的块。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#reverse(java.util.List)'><code>reverse(List)</code></a> | 返回给定 List 的反转视图。注: 如果 List 是不可变的，考虑改用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html#reverse()'><code>ImmutableList.reverse()</code></a> |

```java

List<Integer> countUp = Ints.asList(1, 2, 3, 4, 5);
List<Integer> countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}

List<List<Integer>> parts = Lists.partition(countUp, 2); // {{1, 2}, {3, 4}, {5}}
```

### Static Factories
`Lists`提供以下静态工厂方法：

| 具体实现类型 | 工厂方法                               |
| :------------- | :--------------------------------------- |
| `ArrayList`    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(E...)'>with elements</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayListWithCapacity(int)'>with exact capacity</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayListWithExpectedSize(int)'>with expected size</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newArrayList(java.util.Iterator)'>from <code>Iterator</code></a> |
| `LinkedList`   | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newLinkedList()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Lists.html#newLinkedList(java.lang.Iterable)'>from <code>Iterable</code></a> |

# Sets
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html'><code>Sets</code></a> 工具类包含了若干好用的方法。

### Set-Theoretic Operations
我们提供了许多标准的集合理论操作，作为参数集的视图实现并返回一个 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html'><code>SetView</code></a>，可以使用：
* 直接当作 `Set` 使用，因为它实现了Set接口
* 通过将其复制到另一个可变的集合与 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html#copyInto(S)'><code>copyInto(Set)</code></a> 
    * 通过使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.SetView.html#immutableCopy()'><code>immutableCopy()</code></a>

| 方法                                   |
| :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#union(java.util.Set, java.util.Set)'><code>union(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#intersection(java.util.Set, java.util.Set)'><code>intersection(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#difference(java.util.Set, java.util.Set)'><code>difference(Set, Set)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#symmetricDifference(java.util.Set, java.util.Set)'><code>symmetricDifference(Set, Set)</code></a> |

例如：

```java

Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");

SetView<String> intersection = Sets.intersection(primes, wordsWithPrimeLength); // contains "two", "three", "seven"
// 可以使用交集，但不可变拷贝的读取效率更高
return intersection.immutableCopy();
```

### 其他 Set 工具方法
| 方法                                   | 描述                              | 另请参见                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#cartesianProduct(java.util.List)'><code>cartesianProduct(List&lt;Set&gt;)</code></a> | 返回通过从每个集合中选择一个元素可以获得的每个可能的list。 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#cartesianProduct(java.util.Set...)'><code>cartesianProduct(Set...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#powerSet(java.util.Set)'><code>powerSet(Set)</code></a> | 返回指定集合的所有子集。 |                                          |

```java

Set<String> animals = ImmutableSet.of("gerbil", "hamster");
Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");

Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
// {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
//  {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}

Set<Set<String>> animalSets = Sets.powerSet(animals);
// {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}
```

### Static Factories
`Sets` 提供以下静态工厂方法：

| 具体实现类型  | 工厂方法                                |
| :-------------- | :--------------------------------------- |
| `HashSet`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(E...)'>with elements</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSetWithExpectedSize(int)'>with expected size</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newHashSet(java.util.Iterator)'>from <code>Iterator</code></a> |
| `LinkedHashSet` | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSet(java.lang.Iterable)'>from <code>Iterable</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newLinkedHashSetWithExpectedSize(int)'>with expected size</a> |
| `TreeSet`       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet(java.util.Comparator)'>with <code>Comparator</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Sets.html#newTreeSet(java.lang.Iterable)'>from <code>Iterable</code></a> |

# Maps
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html'><code>Maps</code></a> 类有若干值得单独说明的、很酷的方法。

### `uniqueIndex`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#uniqueIndex(java.lang.Iterable, com.google.common.base.Function)'><code>Maps.uniqueIndex(Iterable, Function)</code></a> 针对一组对象，它们在某个属性上分别有独一无二的值，并希望能够查找基于属性的这些对象的常见情况。

比方说，我们有一堆字符串，这些字符串的长度都是独一无二的，而我们希望能够按照特定长度查找字符串：

```java

ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings, new Function<String, Integer> () {
    public Integer apply(String string) {
      return string.length();
    }
  });
```

如果索引不是唯一的，请参阅下面的Multimaps.index。

### `difference`
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#difference(java.util.Map, java.util.Map)'><code>Maps.difference(Map, Map)</code></a> 用来比较两个 Map 以获取所有不同点。该方法返回`MapDifference`对
象，把不同点的Venn分解为：

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesInCommon()'><code>entriesInCommon()</code></a> | 两个Map中都有的映射项，包括匹配的键与值 |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesDiffering()'><code>entriesDiffering()</code></a> | 键相同但是值不同值映射项。返回的 Map 的值类型为<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.ValueDifference.html'><code>MapDifference.ValueDifference</code></a>，以表示左右两个不同的值 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnLeft()'><code>entriesOnlyOnLeft()</code></a> | 键只存在于左边 Map 的映射项 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnRight()'><code>entriesOnlyOnRight()</code></a> | 键只存在于右边 Map 的映射项 |

```java

Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
MapDifference<String, Integer> diff = Maps.difference(left, right);

diff.entriesInCommon(); // {"b" => 2}
diff.entriesDiffering(); // {"c" => (3, 4)}
diff.entriesOnlyOnLeft(); // {"a" => 1}
diff.entriesOnlyOnRight(); // {"d" => 5}
```

### `BiMap` 工具方法
Guava 中处理 `BiMap` 的工具方法在 `Maps` 类中，因为 `BiMap` 也是一种 `Map` 实现。

| `BiMap` 工具方法 | 相应的 `Map` 工具方法 |
|:----------------|:----------------------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#synchronizedBiMap(com.google.common.collect.BiMap)'><code>synchronizedBiMap(BiMap)</code> <table><thead><th> <code>Collections.synchronizedMap(Map)</code></a> </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#unmodifiableBiMap(com.google.common.collect.BiMap)'><code>unmodifiableBiMap(BiMap)</code> </td><td> <code>Collections.unmodifiableMap(Map)</code></a> </td></tr></tbody></table>

### 静态工厂方法
`Maps`提供了以下静态工厂方法。

| 具体实现类型                           | 工厂方法                                |
| :--------------------------------------- | :--------------------------------------- |
| `HashMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap(java.util.Map)'>from <code>Map</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMapWithExpectedSize(int)'>with expected size</a> |
| `LinkedHashMap`                          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap(java.util.Map)'>from <code>Map</code></a> |
| `TreeMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.Comparator)'>from <code>Comparator</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.SortedMap)'>from <code>SortedMap</code></a> |
| `EnumMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.lang.Class)'>from <code>Class</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.util.Map)'>from <code>Map</code></a> |
| `ConcurrentMap` (支持所有操作) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newConcurrentMap()'>basic</a> |
| `IdentityHashMap`                        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newIdentityHashMap()'>basic</a> |

# Multisets
标准的 `Collection` 操作会忽略 `Multiset` 重复元素的个数，而只关心元素是否存在于 `Multiset` 中，如 `containsAll`
方法。为此，  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html'><code>Multisets</code></a> 提供了若干方法，以顾及 `Multiset` 元素的重复性：

| 方法                                   | 说明                              | 和 `Collection` 方法的区别      |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#containsOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>containsOccurrences(Multiset sup, Multiset sub)</code></a> | 对任意 `o`，如果 sub.count(o)<=super.count(o)，返回`true` | `Collection.containsAll` 忽略个数，而只关心 sub 的元素是否都在 super 中 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#removeOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>removeOccurrences(Multiset removeFrom, Multiset toRemove)</code></a> | 对 toRemove 中的重复元素，仅在 removeFrom 中删除相同个数。| `Collection.removeAll` 移除所有出现在 `toRemove` 的元素 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#retainOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>retainOccurrences(Multiset removeFrom, Multiset toRetain)</code></a> | 修改 `removeFrom`，以保证任意`o`都符合 `removeFrom.count(o) <= toRetain.count(o)` . | `Collection.retainAll` 保留所有出现在 `toRetain` 的元素 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#intersection(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>intersection(Multiset, Multiset)</code></a> | 返回两个 `multiset` 的交集; 非破坏性的替代`retainOccurrences`。 | 没有类似方法                         |

```java

Multiset<String> multiset1 = HashMultiset.create();
multiset1.add("a", 2);

Multiset<String> multiset2 = HashMultiset.create();
multiset2.add("a", 5);

multiset1.containsAll(multiset2); // 返回true；因为包含了所有不重复元素，
  // 虽然multiset1实际上包含2个"a"，而multiset2包含5个"a"
Multisets.containsOccurrences(multiset1, multiset2); // returns false

multiset2.removeOccurrences(multiset1); // multiset2 now contains 3 occurrences of "a"

multiset2.removeAll(multiset1); // multiset2移除所有"a"，虽然multiset1只有2个"a"
multiset2.isEmpty(); // returns true
```

`Multisets` 中的其他工具方法还包括：

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#copyHighestCountFirst(com.google.common.collect.Multiset)'><code>copyHighestCountFirst(Multiset)</code></a> | 返回 Multiset 的不可变拷贝，并将元素按重复
出现的次数做降序排列 |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableMultiset(com.google.common.collect.Multiset)'><code>unmodifiableMultiset(Multiset)</code></a> | 返回 Multiset 的只读视图|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableSortedMultiset(com.google.common.collect.SortedMultiset)'><code>unmodifiableSortedMultiset(SortedMultiset)</code></a> | 返回 SortedMultiset 的只读视图 |

```java

Multiset<String> multiset = HashMultiset.create();
multiset.add("a", 3);
multiset.add("b", 5);
multiset.add("c", 1);

ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);

// highestCountFirst，包括它的entrySet和elementSet，按{"b", "a", "c"}排列元素
```

# Multimaps
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html'><code>Multimaps</code></a> 提供了若干值得单独说明的通用工具方法

### `index`
作为 `Maps.uniqueIndex` 的兄弟方法， <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#index(java.lang.Iterable, com.google.common.base.Function)'><code>Multimaps.index(Iterable, Function)</code></a> 通常针对的场景是：有一组对象，它们有共同的特定属性，我们希望按照这个属性的值查询对象，但属性值不一定是独一无二的。

假设我们要根据字符串的长度对字符串进行分组。

```java

ImmutableSet<String> digits = ImmutableSet.of("zero", "one", "two", "three", "four",
  "five", "six", "seven", "eight", "nine");
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
ImmutableListMultimap<Integer, String> digitsByLength = Multimaps.index(digits, lengthFunction);
/*
 * digitsByLength maps:
 *  3 => {"one", "two", "six"}
 *  4 => {"zero", "four", "five", "nine"}
 *  5 => {"three", "seven", "eight"}
 */
```

### `invertFrom`
由于`Multimap`可以将多个键映射到一个值，并将一个键映射到多个值，因此反转`Multimap`可能很有用。 Guava提供了 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#invertFrom(com.google.common.collect.Multimap, M)'><code>invertFrom(Multimap toInvert, Multimap dest)</code></a> 做这个操作，并且你可以自由选择反转后的 `Multimap` 实现。

注意：如果您使用`ImmutableMultimap`，请考虑使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html#inverse()'><code>ImmutableMultimap.inverse()</code></a> 。

```java

ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
multimap.putAll("b", Ints.asList(2, 4, 6));
multimap.putAll("a", Ints.asList(4, 2, 1));
multimap.putAll("c", Ints.asList(2, 5, 3));

TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap.<String, Integer> create());
// note that we choose the implementation, so if we use a TreeMultimap, we get results in order
/*
 * inverse maps:
 *  1 => {"a"}
 *  2 => {"a", "b", "c"}
 *  3 => {"c"}
 *  4 => {"a", "b"}
 *  5 => {"c"}
 *  6 => {"b"}
 */
```

### `forMap`
想在 Map 对象上使用 `Multimap` 的方法吗？<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#forMap(java.util.Map)'><code>forMap(Map)</code></a> 把 Map 包装成 `SetMultimap`。这个方法特别有用，例如，与 `Multimaps.invertFrom` 结合使用，可以把多对一的 Map 反转为一对多的 `Multimap`。

```java

Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
// multimap maps ["a" => {1}, "b" => {1}, "c" => {2}]
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.<Integer, String> create());
// inverse maps [1 => {"a", "b"}, 2 => {"c"}]
```

### 包装器
`Multimaps` 提供了传统的包装方法，以及让你选择 `Map` 和 `Collection` 类型以自定义 `Multimap` 实现的工具方法。

| 只读包装          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| :-------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| 同步包装          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| 自定义实现  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map, com.google.common.base.Supplier)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSortedSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SortedSetMultimap</code></a> |

自定义 `Multimap` 的方法允许你指定 `Multimap` 中的特定实现。但要注意的是：

* Multimap 假设对 Map 和 Supplier 产生的集合对象有完全所有权。这些自定义对象应避免手动更新，并且在提供给 Multimap 时应该是空的，此外还不应该使用软引用、弱引用或虚引用。
* 无法保证修改了 Multimap 以后，底层 Map 的内容是什么样的。
    * 当任何并发操作更新multimap时，multimap不是线程安全的，即使map和工厂生成的实例也是如此。 并发读操作将正常工作，但如果需要，使用`synchronized` 包装解决此问题。
    * 如果map，factory，factory生成的lists和multimap内容都是可序列化的，multimap是可序列化的。
    * `Multimap.get(key)`返回的集合与Supplier返回的集合的类型不同，但是如果Supplier返回RandomAccess列表，`Multimap.get(key)`返回的列表也将是随机访问。

请注意，用来自定义 `Multimap` 的方法需要一个 `Supplier` 参数，以创建崭新的集合。下面有个实现 `ListMultimap` 的例子——用 `TreeMap` 做映射，而每个键对应的多个值用 `LinkedList` 存储。

```java

ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
  Maps.<String, Collection<Integer>>newTreeMap(),
  new Supplier<LinkedList<Integer>>() {
    public LinkedList<Integer> get() {
      return Lists.newLinkedList();
    }
  });
```

# Tables
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html'><code>Tables</code></a> 类提供了若干称手的工具方法。

### `customTable`
相比`Multimaps.newXXXMultimap(Map, Supplier)`工具方法，<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#newCustomTable(java.util.Map, com.google.common.base.Supplier)'><code>Tables.newCustomTable(Map, Supplier&lt;Map&gt;)</code></a> 允许你指定 Table 用什么样的 map 实现行和列。

```java
// use LinkedHashMaps instead of HashMaps
Table<String, Character, Integer> table = Tables.newCustomTable(
  Maps.<String, Map<Character, Integer>>newLinkedHashMap(),
  new Supplier<Map<Character, Integer>> () {
    public Map<Character, Integer> get() {
      return Maps.newLinkedHashMap();
    }
  });
```

### `transpose`
 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#transpose(com.google.common.collect.Table)'><code>transpose(Table&lt;R, C, V&gt;)</code></a> 方法允许你把`Table<R, C, V>`转置成`Table<C, R, V>`。例如，如果你在用 Table 构建加权有向图，这个方法就可以把有向图反转。

### 包装器
还有很多你熟悉和喜欢的 Table 包装类。然而，在大多数情况下还请使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a> 。

| Unmodifiable | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableTable(com.google.common.collect.Table)'><code>Table</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableRowSortedTable(com.google.common.collect.RowSortedTable)'><code>RowSortedTable</code></a> |
|:-------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
