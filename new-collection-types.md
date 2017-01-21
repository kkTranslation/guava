TODO: More examples

Guava引入了一些JDK中没有的，但我们发现非常有用的新集合类型。 这些新类型是为了和 JDK 集合框架共存，而没有往 JDK 集合抽象中硬塞其他概念。

作为一般规则，Guava集合实现非常精确地遵循JDK接口契约。

# Multiset
统计一个词在文档中出现了多少次，传统的做法是这样的：

```java
Map<String, Integer> counts = new HashMap<String, Integer>();
for (String word : words) {
  Integer count = counts.get(word);
  if (count == null) {
    counts.put(word, 1);
  } else {
    counts.put(word, count + 1);
  }
}
```

这种写法很笨拙，也很容易出错，并且不支持同时收集多种统计数据，如总词数。我们可以做的更好。

Guava提供了一个新的集合类型<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html'><code>Multiset</code></a>，它支持添加多个元素。 维基百科从数学角度这样定义 Multiset：“集合[set]概念的泛化，它的元素可以重复出现…与集合[set]相同而与元组[tuple]相反的是，Multiset 元素的顺序是无关紧要的：Multiset {a, a, b}和{a, b, a}是相等的。

有两种主要的方式来看：

* 没有元素顺序限制的 `ArrayList`：排序并不重要。
* `Map<E, Integer>`，键为元素，值为计数。

Guava的`Multiset` API结合了`Multiset`的两种思考方式，如下：
* 当把 `Multiset` 看成普通的 `Collection` 时，它表现得就像无序的 `ArrayList`：
    * `add(E)`添加单个给定元素
    * `iterator()`返回一个迭代器，包含 Multiset 的所有元素（包括重复的元素）
    * `size()`返回所有元素的总个数（包括重复的元素）
* 当把 `Multiset` 看作 `Map<E, Integer>`时，它也提供了符合性能期望的查询操作：
    * `count(Object)`返回与该元素相关联的计数。 对于`HashMultiset`，count为O(1)，对于`TreeMultiset`，count为O(log n)等。
    * `entrySet()`返回一个`Set<Multiset.Entry>`，它类似于`Map`的`entrySet`。
    * `elementSet()`返回multiset的distinct元素的一个`Set <E>`，类似于`keySet()`。
    * 所有 `Multiset` 实现的内存消耗随着不重复元素的个数线性增长。

值得注意的是，除了极少数情况，`Multiset` 和 JDK 中原有的 Collection 接口契约完全一致——具体来说，`TreeMultiset` 在判断元素是否相等时，与 `TreeSet` 一样用 compare，而不是 `Object.equals`。另外特别注意的是，`Multiset.addAll(Collection)`可以添加 `Collection` 中的所有元素并进行计数，这比用 for 循环往 `Map` 添加元素和计数
方便多了。

| 方法                                   | 描述                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#count(java.lang.Object)'><code>count(E)</code></a> | 给定元素在 Multiset 中的计数。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#elementSet()'><code>elementSet()</code></a> | 将`Multiset <E>`的不重复元素视为`Set <E>`。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#entrySet()'><code>entrySet()</code></a> | 和 Map 的 `entrySet` 类似，返回 `Set<Multiset.Entry<E>>`，其中包含的 Entry 支持 `getElement()`和 `getCount()`方法 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#add(java.lang.Object,int)'><code>add(E, int)</code></a> | 添加指定元素的指定出现次数。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#remove(java.lang.Object, int)'><code>remove(E, int)</code></a> | 删除指定元素的指定出现次数。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#setCount(E, int)'><code>setCount(E, int)</code></a> | 将指定元素的出现次数设置为指定的非负数。 |
| `size()`                                 | 返回集合元素的总个数（包括重复的元素） |

## Multiset Is Not A Map

请注意，`Multiset<E>`不是 `Map<E, Integer>`，虽然 Map 可能是某些 `Multiset` 实现的一部分。准确来说 `Multiset`是一种 `Collection` 类型，并履行了 Collection 接口相关的契约。关于 Multiset 和 Map 的显著区别还包括：

* `Multiset<E>` 中的元素计数只能是正数。任何元素的计数都不能为负数，也不能是 `0`。也不会出现在`elementSet()`或 `entrySet()`的视图中。
* `multiset.size()`返回集合的大小，等同于所有元素计数的总和。对于不重复元素的个数，应使用 `elementSet().size()`方法。（因此，`add(E)`把 `multiset.size()`加 1）
    * `multiset.iterator()`会遍历每个元素的每次出现，因此迭代长度等于 `multiset.size()`。
    * `Multiset` 支持直接添加、删除或设置元素的计数。`setCount(elem, 0)`等同于移除所有元素。
    * 对multiset中没有的元素，`multiset.count(elem)`始终返回 `0`。

## Implementations

Guava 提供了多种 Multiset 的实现，_大致_对应于JDK映射实现。

| Map                 | 对应的Multiset                   | 是否支持`null`元素     |
| :------------------ | :--------------------------------------- | :--------------------------- |
| `HashMap`           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashMultiset.html'><code>HashMultiset</code></a> | 是                          |
| `TreeMap`           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeMultiset.html'><code>TreeMultiset</code></a> | 是（如果 comparator 支持的话） |
| `LinkedHashMap`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultiset.html'><code>LinkedHashMultiset</code></a> | 是                          |
| `ConcurrentHashMap` | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ConcurrentHashMultiset.html'><code>ConcurrentHashMultiset</code></a> | 否                           |
| `ImmutableMap`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html'><code>ImmutableMultiset</code></a> | 否                           |

## SortedMultiset
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/SortedMultiset.html'><code>SortedMultiset</code></a> 是 `Multiset` 接口的变种，它支持高效地获取指定范围的子集。比方说，你可以用 `latencies.subMultiset(0,BoundType.CLOSED, 100, BoundType.OPEN).size()`来统计你的站点中延迟在 100 毫秒以内的访问，然后把这个值和 `latencies.size()`相比，以获取这个延迟水平在总体访问中的比例。

`TreeMultiset`实现了`SortedMultiset`接口。 在撰写本文时，`ImmutableSortedMultiset`仍在测试与GWT的兼容性。

# Multimap
每个经验丰富的Java程序员都在某一点上实现了`Map<K, List<V>>`或`Map<K, Set<V>>`，并处理了该结构的尴尬。 Guava的[Multimap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html)框架可以轻松处理从键到多个值的映射。也可以这么说，`Multimap` 是把键映射到任意多个值的一般方式。

可以用两种方式思考 Multimap 的概念：”键-单个值映射”的集合：

> a -> 1
> a -> 2
> a -> 4
> b -> 3
> c -> 5

或者”键-值集合映射”的映射：

> a -> [1, 2, 4]
> b -> [3]
> c -> [5]

一般来说，`Multimap`接口在第一个视图中是最好的，但允许你使用`asMap()`视图查看它，它返回一个`Map<K, Collection<V>>`。 最重要的是，不会有任何键映射到空集合：键映射到至少一个值，或者它根本不存在于`Multimap`中。 如果您希望能够区分没有映射值的键和不存在的键，则更合适的数据结构可能是[Graph](https://github.com/google/guava/wiki/GraphsExplained)（支持隔离节点）。

然而，您很少直接使用`Multimap`接口; 更多的时候你会使用`ListMultimap`或`SetMultimap`，它们分别将键映射到`List`或`Set`。 


## Modifying

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#get(K)'><code>Multimap.get(key)</code></a> 返回与指定键相关联的值的视图，即使当前没有值。 `ListMultimap`返回一个`List`，`SetMultimap`返回一个`Set`。

修改并写入到底层的`Multimap`。 例如，
```java

Set<Person> aliceChildren = childrenMultimap.get(alice);
aliceChildren.clear();
aliceChildren.add(bob);
aliceChildren.add(carol);
```
反映到底层的`multimap`。

其他（更直接地）修改 Multimap 的方法包括：

| Signature                                | Description                              | Equivalent                               |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#put(K, V)'><code>put(K, V)</code></a> | 添加键到单个值的映射 | `multimap.get(key).add(value)`           |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#putAll(K, java.lang.Iterable)'><code>putAll(K, Iterable&lt;V&gt;)</code></a> | 依次添加键到多个值的映射 | `Iterables.addAll(multimap.get(key), values)` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#remove(java.lang.Object, java.lang.Object)'><code>remove(K, V)</code></a> | 删除一个`key`对应`value`，如果`multimap`改变，返回`true`。 | `multimap.get(key).remove(value)`        |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#removeAll(java.lang.Object)'><code>removeAll(K)</code></a> | 删除并返回与指定键对应的所有值。 返回的集合可以是可修改的，但不会影响multimap。（返回适当的集合类型。） | `multimap.get(key).clear()`              |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#replaceValues(K, java.lang.Iterable)'><code>replaceValues(K, Iterable&lt;V&gt;)</code></a> | 清除所有与`key`对应的值，并设置`key`与每个`values`相关联。返回先前与键相关联的值。 | `multimap.get(key).clear(); Iterables.addAll(multimap.get(key), values)` |

## Views

`Multimap` also supports a number of powerful views.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#asMap()'><code>asMap</code></a> 为 `Multimap<K, V>`提供 `Map<K, Collection<V>>`形式的视图。返回的 `Map `支持 `remove` 操作，并且
会反映到底层的 `Multimap`，但它不支持 `put `或 `putAll` 操作。更重要的是，当你想为 `Multimap` 中没有的
键返回 `null`，而不是一个新的、可写的空集合，你就可以使用 `asMap().get(key)`。（并且你应该可以将`asMap().get(key)`转换为适当的集合类型 -比如 `SetMultimap`的`Set`，`ListMultimap`的`List` - 但这里类型系统不允许`ListMultimap`返回`Map<K, List<V>>`。）
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#entries()'><code>entries</code></a> 返回`Multimap`中所有`entries`的`Collection<Map.Entry<K, V>>`。 （对于`SetMultimap`，这是一个`Set`。）
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#keySet()'><code>keySet</code></a> 用 `Set` 表示 `Multimap` 中所有不同的键。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#keys()'><code>keys</code></a> 用 `Multiset` 表示 `Multimap `中的所有键，每个键重复出现的次数等于它映射的值的个数。可以从这个`Multiset` 中移除元素，但不能做添加操作；移除操作会反映到底层的 `Multimap`。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#values()'><code>values()</code></a> 用一个”扁平”的Collection<V>包含 Multimap 中的所有值，全部为一个集合。这类似于Iterables.concat(multimap.asMap().values())`，但它直接返回了单个 `Collection`，而不像` multimap.asMap().values()`那样是按键区分开的 `Collection`。

## Multimap Is Not A Map
`Multimap<K, V>`不是 `Map<K, Collection<V>>`，虽然某些 `Multimap` 实现中可能使用了map。它们之间的显著区别包括：

* `Multimap.get(key)`总是返回非 null、但是可能为空的集合。这并不意味着`Multimap`为相应的键花费内存创建了集合，而只是提供一个集合视图方便你为键增加映射值
* 如果你更喜欢像 `Map` 那样，为 `Multimap` 中没有的键返回 `null`，请使用 asMap()视图获取一个 `Map<K, Collection<V>>`。（或者用静态方法 `Multimaps.asMap()` 为 `ListMultimap` 返回一个 `Map<K,List<V>>`。对于 `SetMultimap` 和 `SortedSetMultimap`，也有类似的静态方法存在）。
* 当且仅当有值映射到键时，`Multimap.containsKey(key)`才会返回 `true`。尤其需要注意的是，如果键 k 之前映射过一个或多个值，但它们都被移除后，`Multimap.containsKey(key)`会返回 `false`。
* `Multimap.entries()`返回 `Multimap `中所有”键-单个值映射”——包括重复键。如果你想要得到所有”键-值集合映射”，请使用 `asMap().entrySet()`。
* `Multimap.size()`返回所有”键-单个值映射”的数量，而非不同键的数量。使用`Multimap.keySet().size()`来获取不同键的数量。

## Implementations
`Multimap` 提供了多种形式的实现。在大多数要使用 `Map<K, Collection<V>>`的地方，你都可以使用它们：

| 实现                           | 键行为类似... | 值行为类似.. |
| :--------------------------------------- | :------------------ | :------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ArrayListMultimap.html'><code>ArrayListMultimap</code></a> | `HashMap`           | `ArrayList`          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashMultimap.html'><code>HashMultimap</code></a> | `HashMap`           | `HashSet`            |
| <a href="http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedListMultimap.html"><code>LinkedListMultimap</code></a> `*` | `LinkedHashMap``*`  | `LinkedList``*`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultimap.html'><code>LinkedHashMultimap</code></a>`**` | `LinkedHashMap`     | `LinkedHashSet`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeMultimap.html'><code>TreeMultimap</code></a> | `TreeMap`           | `TreeSet`            |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html'><code>ImmutableListMultimap</code></a> | `ImmutableMap`      | `ImmutableList`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html'><code>ImmutableSetMultimap</code></a> | `ImmutableMap`      | `ImmutableSet`       |

除了两个不可变形式的实现，其他所有实现都支持 null 键和 null 值

`*` `LinkedListMultimap.entries()`保留了所有键和值的迭代顺序。详情见 doc 链接。

`**` `LinkedHashMultimap`保留了映射项的插入顺序，包括键插入的顺序，以及键映射的所有值的插入顺序。

注意，并不是所有的 `Multimap` 都和上面列出的一样，使用`Map<K, Collection<V>>`来实现（特别是，一些Multimap实现用了自定义的 hashTable，以最小化开销）

如果你想要更多的定制化，请用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map,%20com.google.common.base.Supplier)'><code>Multimaps.newMultimap(Map, Supplier&lt;Collection&gt;)</code></a> 或 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)'>`list`</a> 和 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)'>`set`</a>  版本，使用自定义的 Collection、List 或 Set 实现 Multimap。

# BiMap
实现键值对的双向映射需要维护两个单独的 map，并使它们保持同步。但这种方式很容易出错，并且当值已经存在于map中时会非常混乱。例如：

```java

Map<String, Integer> nameToId = Maps.newHashMap();
Map<Integer, String> idToName = Maps.newHashMap();

nameToId.put("Bob", 42);
idToName.put(42, "Bob");
// 如果"Bob"和42已经在map中了，会发生什么?
// 如果我们忘了同步两个map，会有诡异的bug发生...
```

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html'><code>BiMap&lt;K, V&gt;</code></a> 是特殊的 Map：

* 允许您使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#inverse()'><code>inverse()</code></a>反转BiMap <V，K>的键值映射
* 确保值是唯一的，使 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#values()'><code>values()</code></a>返回 `Set`

如果您尝试将键映射到已经存在的值，`BiMap.put(key, value)`将抛出IllegalArgumentException。 如果要删除指定值的任何预先存在的entry ，请改用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#forcePut(java.lang.Object,java.lang.Object)'><code>BiMap.forcePut(key, value)</code></a> 。

```java

BiMap<String, Integer> userId = HashBiMap.create();
...

String userForId = userId.inverse().get(id);
```

## BiMap 的实现

| 键–值实现 | 值–键实现 | 对应的BiMap实现                    |
| :----------------- | :----------------- | :--------------------------------------- |
| `HashMap`          | `HashMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashBiMap.html'><code>HashBiMap</code></a> |
| `ImmutableMap`     | `ImmutableMap`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html'><code>ImmutableBiMap</code></a> |
| `EnumMap`          | `EnumMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumBiMap.html'><code>EnumBiMap</code></a> |
| `EnumMap`          | `HashMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumHashBiMap.html'><code>EnumHashBiMap</code></a> |

注：Maps 类中还有一些诸如 synchronizedBiMap 的 BiMap 工具方法.

# Table
```java

Table<DateOfBirth, LastName, PersonalRecord> records = HashBasedTable.create();
records.put(someBirthday, "Schmo", recordA);
records.put(someBirthday, "Doe", recordB);
records.put(otherBirthday, "Doe", recordC);

records.row(someBirthday); // returns a Map mapping "Schmo" to recordA, "Doe" to recordB
records.column("Doe"); // returns a Map mapping someBirthday to recordB, otherBirthday to recordC
```

通常来说，当你想使用多个键做索引的时候，你可能会用类似 `Map<FirstName, Map<LastName, Person>>`的实现，这种方式很难使用。为此，Guava提供了新集合类型  <a href="http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html"><code>Table</code></a>,它有两个支持所有类型
的键：”行”和”列”。 Table 支持多种视图，以便你从各种角度使用它：
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#rowMap()'><code>rowMap()</code></a>, 用 `Map<R, Map<C, V>>`表现 `Table<R, C, V>`。  同样的, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#rowKeySet()'><code>rowKeySet()</code></a> 返回`Set<R>`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#row(R)'><code>row(r)</code></a> 返回非空`Map <C，V>`。 对这个 map 进行的写操作也将写入 Table 中。
    * 提供了类似的列访问方法: <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#columnMap()'><code>columnMap()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#columnKeySet()'><code>columnKeySet()</code></a>, and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#column(C)'><code>column(c)</code></a>.  （基于列的访问比基于行的访问效率稍差。）
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#cellSet()'><code>cellSet()</code></a> 返回一个表的视图作为一组<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.Cell.html'><code>Table.Cell&lt;R, C, V&gt;</code></a>.  `Cell` 很像`Map.Entry`，但区分行和列键。

提供了几个`Table`实现，包括：
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashBasedTable.html'><code>HashBasedTable</code></a>, 它基本上由`HashMap <R，HashMap <C，V >>`（从Guava 20.0开始，由`LinkedHashMap <R，LinkedHashMap <C，V >>`支持）支持。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeBasedTable.html'><code>TreeBasedTable</code></a>, 它基本上由`TreeMap <R，TreeMap <C，V >>`支持。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a>，它基本上由`ImmutableMap <R，ImmutableMap <C，V >>`支持。 （注意：ImmutableTable已经针对更稀疏和更密集的数据集实现了优化。）
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ArrayTable.html'><code>ArrayTable</code></a>, 要求在构造时就指定行和列的大小，本质上由一个二维数组实现，以提升访问速度和密集 Table 的内存利用率。ArrayTable 与其他 Table 的工作原理有点不同，请参见 Javadoc 了解详情。

# ClassToInstanceMap
有时，map的键不是全相同的类型：它们是类型，并且将它们映射到该类型的值。 Guava为此提供了[ClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html) 。

除了扩展`Map`接口之外，`ClassToInstanceMap`还提供了方法 [T getInstance(Class&lt;T&gt;)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html#getInstance(java.lang.Class)) 和 [T putInstance(Class&lt;T&gt;, T)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html#putInstance(java.lang.Class,java.lang.Object))，避免了在执行类型安全时不必要的强制转换。

`ClassToInstanceMap` 有唯一的泛型参数，通常称为 `B`，代表 Map 支持的所有类型的上界。例如：

```java

ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
numberDefaults.putInstance(Integer.class, Integer.valueOf(0));
```

从技术上讲，`ClassToInstanceMap<B>`实现`Map<Class<? extends B>, B>` - 或者换句话说，从B的子类到B的实例的映射。这可以使得包含在ClassToInstanceMap中的通用类型有点混淆，但请记住 `B` 始终是 `Map` 所支持类型的上界——通常 `B` 就是 `Object`。

Guava提供了两种有用的实现：[MutableClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MutableClassToInstanceMap.html) 和 [ImmutableClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html)。

**重要**：像任何其他`Map <Class，Object>`一样，`ClassToInstanceMap`可以包含基本类型的entry，基本类型及其对应的包装类型可以映射到不同的值。
# RangeSet
RangeSet描述了一组不相连的、非空的区间。当把一个区间添加到可变的RangeSet时，所有相连的区间会被合并，并将空区间忽略。例如：

```java

   RangeSet<Integer> rangeSet = TreeRangeSet.create();
   rangeSet.add(Range.closed(1, 10)); // {[1, 10]}
   rangeSet.add(Range.closedOpen(11, 15)); // 不相连区间: {[1, 10], [11, 15)}
   rangeSet.add(Range.closedOpen(15, 20)); // 相连区间; {[1, 10], [11, 20)}
   rangeSet.add(Range.openClosed(0, 0)); // 空区间; {[1, 10], [11, 20)}
   rangeSet.remove(Range.open(5, 10)); // 分割 [1, 10]; {[1, 5], [10, 10], [11, 20)}
```

请注意，要合并 Range.closed(1, 10)和 Range.closedOpen(11, 15)之类的区间，你需要首先用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#canonical(com.google.common.collect.DiscreteDomain)'><code>Range.canonical(DiscreteDomain)</code></a>对区间进行预处理，例如 DiscreteDomain.integers()。

**注意**：GWT不支持`RangeSet`，JDK 1.5后端也不支持; `RangeSet`需要充分使用JDK 1.6中的`NavigableMap`的特性。
## Views

`RangeSet`实现支持非常广泛的视图，包括：

* `complement()`：返回 `RangeSet` 的补码。`complement` 也是 `RangeSet` 类型,包含了不相连的、非空的区间。
* `subRangeSet（Range <C>）`：返回`RangeSet`与指定范围的交集的视图。 这扩展了传统排序集合中的headSet，subSet和tailSet操作。
    * `asRanges()`: 用` Set<Range<C>>`表现 RangeSet，这样可以遍历其中的 Range。
    * `asSet(DiscreteDomain`)（仅 `ImmutableRangeSet` 支持）：用 `ImmutableSortedSet`表现 `RangeSet`，以区间中所有元素的形式而不是区间本身的形式查看。（这个操作不支持 `DiscreteDomain` 和 `RangeSet` 都没有上边界，或都没有下边界的情况）

## Queries

为了方便操作，RangeSet 直接提供了若干查询方法，其中最突出的有:

* `contains(C)`：`RangeSet` 最基本的操作，判断 `RangeSet` 中是否有任何区间包含给定元素。
* `rangeContaining(C)`: 返回包含指定元素的`Range`，如果没有，则返回null。
    * `encloses(Range<C>)`: 简单明了，测试`RangeSet`中的`Range`是否包含指定的区间。
    * `span()`: 返回包括 `RangeSet `中所有区间的最小区间。

# RangeMap
`RangeMap` 描述了”不相交的、非空的区间”到特定值的映射。和 `RangeSet` 不同，`RangeMap` 不会合并相邻的映射，即便相邻的区间映射到相同的值。例如：

```java

RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
rangeMap.put(Range.closed(1, 10), "foo"); // {[1, 10] => "foo"}
rangeMap.put(Range.open(3, 6), "bar"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo"}
rangeMap.put(Range.open(10, 20), "foo"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo", (10, 20) => "foo"}
rangeMap.remove(Range.closed(5, 11)); // {[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"}
```

## Views
`RangeMap`提供两个视图：

* `asMapOfRanges()`: 用 `Map<Range<K>, V>`表现 `RangeMap`。这可以用来遍历 `RangeMap`。
* `subRangeMap(Range<K>)` 用 `RangeMap `类型返回 `RangeMap` 与给定 `Range` 的交集视图。这扩展了传统的 `headMap`、`subMap` 和 `tailMap` 操作。
