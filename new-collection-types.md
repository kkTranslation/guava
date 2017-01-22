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
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#entrySet()'><code>entrySet()</code></a> | 和 Map 的 `entrySet` 类似，返回 `Set<Multiset.Entry<E>>`，其中包含的 Entry 支
持 `getElement()`和 `getCount()`方法 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#add(java.lang.Object,int)'><code>add(E, int)</code></a> | 添加指定元素的指定出现次数。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#remove(java.lang.Object, int)'><code>remove(E, int)</code></a> | 删除指定元素的指定出现次数。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multiset.html#setCount(E, int)'><code>setCount(E, int)</code></a> | 将指定元素的出现次数设置为指定的非负数。 |
| `size()`                                 | 返回集合元素的总个数（包括重复的元素） |

## Multiset Is Not A Map

请注意，`Multiset<E>`不是 `Map<E, Integer>`，虽然 Map 可能是某些 `Multiset` 实现的一部分。准确来说 `Multiset`是一种 `Collection` 类型，并履行了 Collection 接口相关的契约。关于 Multiset 和 Map 的显著区别还包括：

* `Multiset<E>` 中的元素计数只能是正数。任何元素的计数都不能为负数，也不能是 `0`。也不会出现在`elementSet()`或 

`entrySet()`的视图中。
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

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#get(K)'><code>Multimap.get(key)</code></a> returns a _view_ of the values associated with the specified key, even if there are none currently.  For a `ListMultimap`, it returns a `List`, for a `SetMultimap`, it returns a `Set`.

Modifications write through to the underlying `Multimap`.  For example,
```java

Set<Person> aliceChildren = childrenMultimap.get(alice);
aliceChildren.clear();
aliceChildren.add(bob);
aliceChildren.add(carol);
```
writes through to the underlying multimap.

Other ways of modifying the multimap (more directly) include:

| Signature                                | Description                              | Equivalent                               |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#put(K, V)'><code>put(K, V)</code></a> | Adds an association from the key to the value | `multimap.get(key).add(value)`           |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#putAll(K, java.lang.Iterable)'><code>putAll(K, Iterable&lt;V&gt;)</code></a> | Adds associations from the key to each of the values in turn | `Iterables.addAll(multimap.get(key), values)` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#remove(java.lang.Object, java.lang.Object)'><code>remove(K, V)</code></a> | Removes one association from `key` to `value` and returns `true` if the multimap changed. | `multimap.get(key).remove(value)`        |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#removeAll(java.lang.Object)'><code>removeAll(K)</code></a> | Removes and returns all the values associated with the specified key.  The returned collection may or may not be modifiable, but modifying it will not affect the multimap.  (Returns the appropriate collection type.) | `multimap.get(key).clear()`              |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#replaceValues(K, java.lang.Iterable)'><code>replaceValues(K, Iterable&lt;V&gt;)</code></a> | Clears all the values associated with `key` and sets `key` to be associated with each of `values`.  Returns the values that were previously associated with the key. | `multimap.get(key).clear(); Iterables.addAll(multimap.get(key), values)` |

## Views

`Multimap` also supports a number of powerful views.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#asMap()'><code>asMap</code></a> 为 `Multimap<K, V>`提供 `Map<K, Collection<V>>`形式的视图。返回的 `Map `支持 `remove` 操作，并且
会反映到底层的 `Multimap`，但它不支持 `put `或 `putAll` 操作。更重要的是，当你想为 `Multimap` 中没有的
键返回 `null`，而不是一个新的、可写的空集合，你就可以使用 `asMap().get(key)`。（并且你应该可以将`asMap().get(key)`转换为适当的集合类型 -比如 `SetMultimap`的`Set`，`ListMultimap`的`List` - 但这里类型系统不允许`ListMultimap`返回`Map<K, List<V>>`。）
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#entries()'><code>entries</code></a> entries 用Multimap中所有entries 的Collection<Map.Entry<K, V>> 。 （对于SetMultimap，这是一个集合。）
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#keySet()'><code>keySet</code></a> views the distinct keys in the `Multimap` as a `Set`.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#keys()'><code>keys</code></a> views the keys of the `Multimap` as a `Multiset`, with multiplicity equal to the number of values associated to that key.  Elements can be removed from the `Multiset`, but not added; changes will write through.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimap.html#values()'><code>values()</code></a> views the all the values in the `Multimap` as a "flattened" `Collection<V>`, all as one collection.  This is similar to `Iterables.concat(multimap.asMap().values())`, but returns a full `Collection` instead.

## Multimap Is Not A Map
A `Multimap<K, V>` is _not_ a `Map<K, Collection<V>>`, though such a map might be used in a `Multimap` implementation.  Notable differences include:

* `Multimap.get(key)` always returns a non-null, possibly empty collection.  This doesn't imply that the multimap spends any memory associated with the key, but instead, the returned collection is a view that allows you to add associations with the key if you like.
* If you prefer the more `Map`-like behavior of returning `null` for keys that aren't in the multimap, use the `asMap()` view to get a `Map<K, Collection<V>>`.  (Or, to get a `Map<K, `**`List`**`<V>>` from a `ListMultimap`, use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#asMap%28com.google.common.collect.ListMultimap%29'>the static <code>Multimaps.asMap()</code> method</a>. Similar methods exist for `SetMultimap` and `SortedSetMultimap`.)
    * `Multimap.containsKey(key)` is true if and only if there are any elements associated with the specified key.  In particular, if a key `k` was previously associated with one or more values which have since been removed from the multimap, `Multimap.containsKey(k)` will return false.
    * `Multimap.entries()` returns all entries for all keys in the `Multimap`.  If you want all key-collection entries, use `asMap().entrySet()`.
    * `Multimap.size()` returns the number of entries in the entire multimap, not the number of distinct keys.  Use `Multimap.keySet().size()` instead to get the number of distinct keys.

## Implementations
`Multimap` provides a wide variety of implementations.  You can use it in most places you would have used a `Map<K, Collection<V>>`.

| Implementation                           | Keys behave like... | Values behave like.. |
| :--------------------------------------- | :------------------ | :------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ArrayListMultimap.html'><code>ArrayListMultimap</code></a> | `HashMap`           | `ArrayList`          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashMultimap.html'><code>HashMultimap</code></a> | `HashMap`           | `HashSet`            |
| <a href="http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedListMultimap.html"><code>LinkedListMultimap</code></a> `*` | `LinkedHashMap``*`  | `LinkedList``*`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/LinkedHashMultimap.html'><code>LinkedHashMultimap</code></a>`**` | `LinkedHashMap`     | `LinkedHashSet`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeMultimap.html'><code>TreeMultimap</code></a> | `TreeMap`           | `TreeSet`            |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html'><code>ImmutableListMultimap</code></a> | `ImmutableMap`      | `ImmutableList`      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html'><code>ImmutableSetMultimap</code></a> | `ImmutableMap`      | `ImmutableSet`       |

Each of these implementations, except the immutable ones, support null keys and values.

`*` `LinkedListMultimap.entries()` preserves iteration order across non-distinct key values.  See the link for details.

`**` `LinkedHashMultimap` preserves insertion order of entries, as well as the insertion order of keys, and the set of values associated with any one key.

Be aware that not all implementations are actually implemented as a `Map<K, Collection<V>>` with the listed implementations!  (In particular, several `Multimap` implementations use custom hash tables to minimize overhead.)

If you need more customization, use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newMultimap(java.util.Map,%20com.google.common.base.Supplier)'><code>Multimaps.newMultimap(Map, Supplier&lt;Collection&gt;)</code></a> or the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newListMultimap(java.util.Map, com.google.common.base.Supplier)'>list</a> and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Multimaps.html#newSetMultimap(java.util.Map, com.google.common.base.Supplier)'>set</a> versions to use a custom collection, list, or set implementation to back your multimap.

# BiMap
The traditional way to map values back to keys is to maintain two separate maps and keep them both in sync, but this is bug-prone and can get extremely confusing when a value is already present in the map.  For example:

```java

Map<String, Integer> nameToId = Maps.newHashMap();
Map<Integer, String> idToName = Maps.newHashMap();

nameToId.put("Bob", 42);
idToName.put(42, "Bob");
// what happens if "Bob" or 42 are already present?
// weird bugs can arise if we forget to keep these in sync...
```

A <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html'><code>BiMap&lt;K, V&gt;</code></a> is a `Map<K, V>` that

* allows you to view the "inverse" `BiMap<V, K>` with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#inverse()'><code>inverse()</code></a>
* ensures that values are unique, making <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#values()'><code>values()</code></a> a `Set`

`BiMap.put(key, value)` will throw an `IllegalArgumentException` if you attempt to map a key to an already-present value.  If you wish to delete any preexisting entry with the specified value, use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/BiMap.html#forcePut(java.lang.Object,java.lang.Object)'><code>BiMap.forcePut(key, value)</code></a> instead.

```java

BiMap<String, Integer> userId = HashBiMap.create();
...

String userForId = userId.inverse().get(id);
```

## Implementations

| Key-Value Map Impl | Value-Key Map Impl | Corresponding `BiMap`                    |
| :----------------- | :----------------- | :--------------------------------------- |
| `HashMap`          | `HashMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashBiMap.html'><code>HashBiMap</code></a> |
| `ImmutableMap`     | `ImmutableMap`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html'><code>ImmutableBiMap</code></a> |
| `EnumMap`          | `EnumMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumBiMap.html'><code>EnumBiMap</code></a> |
| `EnumMap`          | `HashMap`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/EnumHashBiMap.html'><code>EnumHashBiMap</code></a> |

_Note:_ `BiMap` utilities like `synchronizedBiMap` live in `Maps`.

# Table
```java

Table<DateOfBirth, LastName, PersonalRecord> records = HashBasedTable.create();
records.put(someBirthday, "Schmo", recordA);
records.put(someBirthday, "Doe", recordB);
records.put(otherBirthday, "Doe", recordC);

records.row(someBirthday); // returns a Map mapping "Schmo" to recordA, "Doe" to recordB
records.column("Doe"); // returns a Map mapping someBirthday to recordB, otherBirthday to recordC
```

Typically, when you are trying to index on more than one key at a time, you will wind up with something like `Map<FirstName, Map<LastName, Person>>`, which is ugly and awkward to use.  Guava provides a new collection type, <a href="http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html"><code>Table</code></a>, which supports this use case for any "row" type and "column" type.  `Table` supports a number of views to let you use the data from any angle, including
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#rowMap()'><code>rowMap()</code></a>, which views a `Table<R, C, V>` as a `Map<R, Map<C, V>>`.  Similarly, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#rowKeySet()'><code>rowKeySet()</code></a> returns a `Set<R>`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#row(R)'><code>row(r)</code></a> returns a non-null `Map<C, V>`.  Writes to the `Map` will write through to the underlying `Table`.
    * Analogous column methods are provided: <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#columnMap()'><code>columnMap()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#columnKeySet()'><code>columnKeySet()</code></a>, and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#column(C)'><code>column(c)</code></a>.  (Column-based access is somewhat less efficient than row-based access.)
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.html#cellSet()'><code>cellSet()</code></a> returns a view of the `Table` as a set of <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Table.Cell.html'><code>Table.Cell&lt;R, C, V&gt;</code></a>.  `Cell` is much like `Map.Entry`, but distinguishes the row and column keys.

Several `Table` implementations are provided, including:
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/HashBasedTable.html'><code>HashBasedTable</code></a>, which is essentially backed by a `HashMap<R, HashMap<C, V>>` (as of Guava 20.0, it is backed by a `LinkedHashMap<R, LinkedHashMap<C, V>>`).
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/TreeBasedTable.html'><code>TreeBasedTable</code></a>, which is essentially backed by a `TreeMap<R, TreeMap<C, V>>`.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a>, which is essentially backed by an `ImmutableMap<R, ImmutableMap<C, V>>`.  (Note: `ImmutableTable` has optimized implementations for sparser and denser data sets.)
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ArrayTable.html'><code>ArrayTable</code></a>, which requires that the complete universe of rows and columns be specified at construction time, but is backed by a two-dimensional array to improve speed and memory efficiency when the table is dense.  `ArrayTable` works somewhat differently from other implementations; consult the Javadoc for details.

# ClassToInstanceMap
Sometimes, your map keys aren't all of the same type: they _are_ types, and you want to map them to values of that type.  Guava provides [ClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html) for this purpose.

In addition to extending the `Map` interface, `ClassToInstanceMap` provides the methods [T getInstance(Class&lt;T&gt;)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html#getInstance(java.lang.Class)) and [T putInstance(Class&lt;T&gt;, T)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ClassToInstanceMap.html#putInstance(java.lang.Class,java.lang.Object)), which eliminate the need for unpleasant casting while enforcing type safety.

`ClassToInstanceMap` has a single type parameter, typically named `B`, representing the upper bound on the types managed by the map.  For example:

```java

ClassToInstanceMap<Number> numberDefaults = MutableClassToInstanceMap.create();
numberDefaults.putInstance(Integer.class, Integer.valueOf(0));
```

Technically, `ClassToInstanceMap<B>` implements `Map<Class<? extends B>, B>` -- or in other words, a map from subclasses of B to instances of B.  This can make the generic types involved in `ClassToInstanceMap` mildly confusing, but just remember that `B` is always the upper bound on the types in the map -- usually, `B` is just `Object`.

Guava provides implementations helpfully named [MutableClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/MutableClassToInstanceMap.html) and [ImmutableClassToInstanceMap](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html).

**Important**: Like any other `Map<Class, Object>`, a `ClassToInstanceMap` may contain entries for primitive types, and a primitive type and its corresponding wrapper type may map to different values.

# RangeSet
A `RangeSet` describes a set of _disconnected, nonempty_ ranges.  When adding a range to a mutable `RangeSet`, any connected ranges are merged together, and empty ranges are ignored.  For example:

```java

   RangeSet<Integer> rangeSet = TreeRangeSet.create();
   rangeSet.add(Range.closed(1, 10)); // {[1, 10]}
   rangeSet.add(Range.closedOpen(11, 15)); // disconnected range: {[1, 10], [11, 15)}
   rangeSet.add(Range.closedOpen(15, 20)); // connected range; {[1, 10], [11, 20)}
   rangeSet.add(Range.openClosed(0, 0)); // empty range; {[1, 10], [11, 20)}
   rangeSet.remove(Range.open(5, 10)); // splits [1, 10]; {[1, 5], [10, 10], [11, 20)}
```

Note that to merge ranges like `Range.closed(1, 10)` and `Range.closedOpen(11, 15)`, you must first preprocess ranges with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Range.html#canonical(com.google.common.collect.DiscreteDomain)'><code>Range.canonical(DiscreteDomain)</code></a>, e.g. with `DiscreteDomain.integers()`.

**NOTE**: `RangeSet` is not supported under GWT, nor in the JDK 1.5 backport; `RangeSet` requires full use of the `NavigableMap` features in JDK 1.6.

## Views

`RangeSet` implementations support an extremely wide range of views, including:

* `complement()`: views the complement of the `RangeSet`.  `complement` is also a `RangeSet`, as it contains disconnected, nonempty ranges.
* `subRangeSet(Range<C>)`: returns a view of the intersection of the `RangeSet` with the specified `Range`.  This generalizes the `headSet`, `subSet`, and `tailSet` views of traditional sorted collections.
    * `asRanges()`: views the `RangeSet` as a `Set<Range<C>>` which can be iterated over.
    * `asSet(DiscreteDomain<C>)` (`ImmutableRangeSet` only): Views the `RangeSet<C>` as an `ImmutableSortedSet<C>`, viewing the elements in the ranges instead of the ranges themselves.  (This operation is unsupported if the `DiscreteDomain` and the `RangeSet` are both unbounded above or both unbounded below.)

## Queries

In addition to operations on its views, `RangeSet` supports several query operations directly, the most prominent of which are:

* `contains(C)`: the most fundamental operation on a `RangeSet`, querying if any range in the `RangeSet` contains the specified element.
* `rangeContaining(C)`: returns the `Range` which encloses the specified element, or `null` if there is none.
    * `encloses(Range<C>)`: straightforwardly enough, tests if any `Range` in the `RangeSet` encloses the specified range.
    * `span()`: returns the minimal `Range` that `encloses` every range in this `RangeSet`.

# RangeMap
`RangeMap` is a collection type describing a mapping from disjoint, nonempty ranges to values.  Unlike `RangeSet`, `RangeMap` never "coalesces" adjacent mappings, even if adjacent ranges are mapped to the same values.  For example:

```java

RangeMap<Integer, String> rangeMap = TreeRangeMap.create();
rangeMap.put(Range.closed(1, 10), "foo"); // {[1, 10] => "foo"}
rangeMap.put(Range.open(3, 6), "bar"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo"}
rangeMap.put(Range.open(10, 20), "foo"); // {[1, 3] => "foo", (3, 6) => "bar", [6, 10] => "foo", (10, 20) => "foo"}
rangeMap.remove(Range.closed(5, 11)); // {[1, 3] => "foo", (3, 5) => "bar", (11, 20) => "foo"}
```

## Views
`RangeMap` provides two views:

* `asMapOfRanges()`: views the `RangeMap` as a `Map<Range<K>, V>`.  This can be used, for example, to iterate over the `RangeMap`.
* `subRangeMap(Range<K>)` views the intersection of the `RangeMap` with the specified `Range` as a `RangeMap`.  This generalizes the traditional `headMap`, `subMap`, and `tailMap` operations.
