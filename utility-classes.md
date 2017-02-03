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
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#difference(java.util.Map, java.util.Map)'><code>Maps.difference(Map, Map)</code></a> allows you to compare all the differences between two maps.  It returns a `MapDifference` object, which breaks down the Venn diagram into:

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesInCommon()'><code>entriesInCommon()</code></a> | The entries which are in both maps, with both matching keys and values. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesDiffering()'><code>entriesDiffering()</code></a> | The entries with the same keys, but differing values.  The values in this map are of type <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.ValueDifference.html'><code>MapDifference.ValueDifference</code></a>, which lets you look at the left and right values. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnLeft()'><code>entriesOnlyOnLeft()</code></a> | Returns the entries whose keys are in the left but not in the right map. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MapDifference.html#entriesOnlyOnRight()'><code>entriesOnlyOnRight()</code></a> | Returns the entries whose keys are in the right but not in the left map. |

```java

Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
Map<String, Integer> right = ImmutableMap.of("b", 2, "c", 4, "d", 5);
MapDifference<String, Integer> diff = Maps.difference(left, right);

diff.entriesInCommon(); // {"b" => 2}
diff.entriesDiffering(); // {"c" => (3, 4)}
diff.entriesOnlyOnLeft(); // {"a" => 1}
diff.entriesOnlyOnRight(); // {"d" => 5}
```

### `BiMap` utilities
The Guava utilities on `BiMap` live in the `Maps` class, since a `BiMap`
is also a `Map`.

| `BiMap` utility | Corresponding `Map` utility |
|:----------------|:----------------------------|
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#synchronizedBiMap(com.google.common.collect.BiMap)'><code>synchronizedBiMap(BiMap)</code> <table><thead><th> <code>Collections.synchronizedMap(Map)</code></a> </th></thead><tbody>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#unmodifiableBiMap(com.google.common.collect.BiMap)'><code>unmodifiableBiMap(BiMap)</code> </td><td> <code>Collections.unmodifiableMap(Map)</code></a> </td></tr></tbody></table>

### Static Factories
`Maps` provides the following static factory methods.

| Implementation                           | Factories                                |
| :--------------------------------------- | :--------------------------------------- |
| `HashMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMap(java.util.Map)'>from <code>Map</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newHashMapWithExpectedSize(int)'>with expected size</a> |
| `LinkedHashMap`                          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newLinkedHashMap(java.util.Map)'>from <code>Map</code></a> |
| `TreeMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap()'>basic</a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.Comparator)'>from <code>Comparator</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newTreeMap(java.util.SortedMap)'>from <code>SortedMap</code></a> |
| `EnumMap`                                | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.lang.Class)'>from <code>Class</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newEnumMap(java.util.Map)'>from <code>Map</code></a> |
| `ConcurrentMap` (supporting all operations) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newConcurrentMap()'>basic</a> |
| `IdentityHashMap`                        | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Maps.html#newIdentityHashMap()'>basic</a> |

# Multisets
Standard `Collection` operations, such as `containsAll`, ignore the count of elements in the multiset, and only care about whether elements are in the multiset at all, or not.  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html'><code>Multisets</code></a> provides a number of operations that take into account element multiplicities in multisets.

| Method                                   | Explanation                              | Difference from `Collection` method      |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#containsOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>containsOccurrences(Multiset sup, Multiset sub)</code></a> | Returns `true` if `sub.count(o) <= super.count(o)` for all `o`. | `Collection.containsAll` ignores counts, and only tests whether elements are contained at all. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#removeOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>removeOccurrences(Multiset removeFrom, Multiset toRemove)</code></a> | Removes one occurrence in `removeFrom` for each occurrence of an element in `toRemove`. | `Collection.removeAll` removes all occurences of any element that occurs even once in `toRemove`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#retainOccurrences(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>retainOccurrences(Multiset removeFrom, Multiset toRetain)</code></a> | Guarantees that `removeFrom.count(o) <= toRetain.count(o)` for all `o`. | `Collection.retainAll` keeps all occurrences of elements that occur even once in `toRetain`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#intersection(com.google.common.collect.Multiset, com.google.common.collect.Multiset)'><code>intersection(Multiset, Multiset)</code></a> | Returns a view of the intersection of two multisets; a nondestructive alternative to `retainOccurrences`. | Has no analogue                          |

```java

Multiset<String> multiset1 = HashMultiset.create();
multiset1.add("a", 2);

Multiset<String> multiset2 = HashMultiset.create();
multiset2.add("a", 5);

multiset1.containsAll(multiset2); // returns true: all unique elements are contained, 
  // even though multiset1.count("a") == 2 < multiset2.count("a") == 5
Multisets.containsOccurrences(multiset1, multiset2); // returns false

multiset2.removeOccurrences(multiset1); // multiset2 now contains 3 occurrences of "a"

multiset2.removeAll(multiset1); // removes all occurrences of "a" from multiset2, even though multiset1.count("a") == 2
multiset2.isEmpty(); // returns true
```

Other utilities in `Multisets` include:

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#copyHighestCountFirst(com.google.common.collect.Multiset)'><code>copyHighestCountFirst(Multiset)</code></a> | Returns an immutable copy of the multiset that iterates over elements in descending frequency order. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableMultiset(com.google.common.collect.Multiset)'><code>unmodifiableMultiset(Multiset)</code></a> | Returns an unmodifiable view of the multiset. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multisets.html#unmodifiableSortedMultiset(com.google.common.collect.SortedMultiset)'><code>unmodifiableSortedMultiset(SortedMultiset)</code></a> | Returns an unmodifiable view of the sorted multiset. |

```java

Multiset<String> multiset = HashMultiset.create();
multiset.add("a", 3);
multiset.add("b", 5);
multiset.add("c", 1);

ImmutableMultiset<String> highestCountFirst = Multisets.copyHighestCountFirst(multiset);

// highestCountFirst, like its entrySet and elementSet, iterates over the elements in order {"b", "a", "c"}
```

# Multimaps
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html'><code>Multimaps</code></a> provides a number of general utility operations that deserve individual explanation.

### `index`
The cousin to `Maps.uniqueIndex`, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#index(java.lang.Iterable, com.google.common.base.Function)'><code>Multimaps.index(Iterable, Function)</code></a> answers the case when you want to be able to look up all objects with some particular attribute in common, which is not necessarily unique.

Let's say we want to group strings based on their length.

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
Since `Multimap` can map many keys to one value, and one key to many values, it can be useful to invert a `Multimap`.  Guava provides <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#invertFrom(com.google.common.collect.Multimap, M)'><code>invertFrom(Multimap toInvert, Multimap dest)</code></a> to let you do this, without choosing an implementation for you.

_NOTE:_ If you are using an `ImmutableMultimap`, consider <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html#inverse()'><code>ImmutableMultimap.inverse()</code></a> instead.

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
Need to use a `Multimap` method on a `Map`?  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#forMap(java.util.Map)'><code>forMap(Map)</code></a> views a `Map` as a `SetMultimap`.  This is particularly useful, for example, in combination with `Multimaps.invertFrom`.

```java

Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
// multimap maps ["a" => {1}, "b" => {1}, "c" => {2}]
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap.<Integer, String> create());
// inverse maps [1 => {"a", "b"}, 2 => {"c"}]
```

### Wrappers
`Multimaps` provides the traditional wrapper methods, as well as tools to get custom `Multimap` implementations based on `Map` and `Collection` implementations of your choice.

| Unmodifiable          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#unmodifiableSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| :-------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| Synchronized          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedMultimap(com.google.common.collect.Multimap)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedListMultimap(com.google.common.collect.ListMultimap)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSetMultimap(com.google.common.collect.SetMultimap)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#synchronizedSortedSetMultimap(com.google.common.collect.SortedSetMultimap)'><code>SortedSetMultimap</code></a> |
| Custom Implementation | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map, com.google.common.base.Supplier)'><code>Multimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)'><code>ListMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SetMultimap</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSortedSetMultimap(java.util.Map, com.google.common.base.Supplier)'><code>SortedSetMultimap</code></a> |

The custom `Multimap` implementations let you specify a particular implementation that should be used in the returned `Multimap`.  Caveats include:

* The multimap assumes complete ownership over of map and the lists returned by factory. Those objects should not be manually updated, they should be empty when provided, and they should not use soft, weak, or phantom references.
* **No guarantees are made** on what the contents of the `Map` will look like after you modify the `Multimap`.
    * The multimap is not threadsafe when any concurrent operations update the multimap, even if map and the instances generated by factory are. Concurrent read operations will work correctly, though.  Work around this with the `synchronized` wrappers if necessary.
    * The multimap is serializable if map, factory, the lists generated by factory, and the multimap contents are all serializable.
    * The collections returned by `Multimap.get(key)` are _not_ of the same type as the collections returned by your `Supplier`, though if you supplier returns `RandomAccess` lists, the lists returned by `Multimap.get(key)` will also be random access.

Note that the custom `Multimap` methods expect a `Supplier` argument to generate fresh new collections.  Here is an example of writing a `ListMultimap` backed by a `TreeMap` mapping to `LinkedList`.

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
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html'><code>Tables</code></a> class provides a few handy utilities.

### `customTable`
Comparable to the `Multimaps.newXXXMultimap(Map, Supplier)` utilities, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#newCustomTable(java.util.Map, com.google.common.base.Supplier)'><code>Tables.newCustomTable(Map, Supplier&lt;Map&gt;)</code></a> allows you to specify a `Table` implementation using whatever row or column map you like.

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
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#transpose(com.google.common.collect.Table)'><code>transpose(Table&lt;R, C, V&gt;)</code></a> method allows you to view a `Table<R, C, V>` as a `Table<C, R, V>`.

### Wrappers
These are the familiar unmodifiability wrappers you know and love.  Consider, however, using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a> instead in most cases.

| Unmodifiable | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableTable(com.google.common.collect.Table)'><code>Table</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Tables.html#unmodifiableRowSortedTable(com.google.common.collect.RowSortedTable)'><code>RowSortedTable</code></a> |
|:-------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
