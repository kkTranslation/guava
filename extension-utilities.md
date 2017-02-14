# Introduction
有时候你需要实现自己的集合扩展。也许你想要在元素被添加到列表时增加特定的行为，或者你想实现一个 `Iterable`，其底层实际上是遍历数据库查询的结果集。Guava 为你，也为我们自己提供了若干工具方法，以便让类似的工作变得更简单。（毕竟，我们自己也要用这些工具扩展集合框架。）

# Forwarding装饰器
针对所有类型的集合接口，Guava 都提供了 Forwarding 抽象类以简化 <a href='http://en.wikipedia.org/wiki/Decorator_pattern'>decorator pattern</a>的使用。

Forwarding 抽象类定义了一个抽象方法：`delegate()`，你可以覆盖这个方法来返回被装饰对象。所有其他方法都会直接委托给 `delegate()`。例如说：`ForwardingList.get(int)`实际上执行了 `delegate().get(int)`。

通过创建 `ForwardingXXX` 的子类并实现 `delegate()`方法，可以选择性地覆盖子类的方法来增加装饰功能，而不需要自己委托每个方法——译者注：因为所有方法都默认委托给 `delegate()`返回的对象，你可以只覆盖需要装饰的方法。

此外，许多方法都有一个`standardMethod`实现，可以用来恢复被装饰对象的默认行为，以提供相同的优点。扩展`AbstractList`或JDK中的其他框架类。

让我们看看这个例子。假定你想装饰一个 List，让其记录所有添加进来的元素。当然，无论元素是用什么方法——`add(int, E)`, `add(E)`, 或 `addAll(Collection)`——添加进来的，我们都希望进行记录，因此我们需要覆盖所有这些方法。

```java
class AddLoggingList<E> extends ForwardingList<E> {

  final List<E> delegate; // backing list
  
  @Override 
  protected List<E> delegate() {
    return delegate;
  }
  @Override 
  public void add(int index, E elem) {
    log(index, elem);
    super.add(index, elem);
  }
  @Override 
  public boolean add(E elem) {
    return standardAdd(elem); // 用add(int, E)实现
  }
  @Override 
  public boolean addAll(Collection<? extends E> c) {
    return standardAddAll(c); // 用add实现
  }
}
```

记住，默认情况下，所有方法都直接转发到被代理对象，因此覆盖 `ForwardingMap.put` 并不会改变 `ForwardingMap.putAll` 的行为。小心覆盖所有需要改变行为的方法，并且确保装饰后的集合满足接口契约。

通常，抽象集合框架提供的大多数方法（如`AbstractList`）也作为`Forwarding`装饰器中的标准实现提供。

对提供特定视图的接口，`Forwarding` 装饰器也为这些视图提供了相应的”标准方法”实现。例如，`ForwardingMap` 提供 `StandardKeySet`、`StandardValues` 和 `StandardEntrySet` 类，它们在可以的情况下都会把自己的方法委托给被装饰的 `Map`，把不能委托的声明为抽象方法。

| 接口       | 反转装饰                    |
| :-------------- | :--------------------------------------- |
| `Collection`    | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingCollection.html'><code>ForwardingCollection</code></a> |
| `List`          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingList.html'><code>ForwardingList</code></a> |
| `Set`           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingSet.html'><code>ForwardingSet</code></a> |
| `SortedSet`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingSortedSet.html'><code>ForwardingSortedSet</code></a> |
| `Map`           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingMap.html'><code>ForwardingMap</code></a> |
| `SortedMap`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingSortedMap.html'><code>ForwardingSortedMap</code></a> |
| `ConcurrentMap` | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingConcurrentMap.html'><code>ForwardingConcurrentMap</code></a> |
| `Map.Entry`     | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingMapEntry.html'><code>ForwardingMapEntry</code></a> |
| `Queue`         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingQueue.html'><code>ForwardingQueue</code></a> |
| `Iterator`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingIterator.html'><code>ForwardingIterator</code></a> |
| `ListIterator`  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingListIterator.html'><code>ForwardingListIterator</code></a> |
| `Multiset`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingMultiset.html'><code>ForwardingMultiset</code></a> |
| `Multimap`      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingMultimap.html'><code>ForwardingMultimap</code></a> |
| `ListMultimap`  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingListMultimap.html'><code>ForwardingListMultimap</code></a> |
| `SetMultimap`   | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ForwardingSetMultimap.html'><code>ForwardingSetMultimap</code></a> |

# PeekingIterator
有时，普通的 `Iterator` 接口还不够。

`Iterators` 提供一个 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/Iterators.html#peekingIterator(java.util.Iterator)'><code>Iterators.peekingIterator(Iterator)</code></a>, 方法，来把 `Iterator` 包装为 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/PeekingIterator.html'><code>PeekingIterator</code></a>这是 `Iterator` 的子类，它能让你事先窥视 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/PeekingIterator.html#peek()'><code>peek()</code></a> 到下一次调用 `next()`返回的元素。

注意：`Iterators.peekingIterator`返回的`PeekingIterator`不支持`peek()`之后的`remove()`调用。

举个例子：复制一个 `List`，并去除连续的重复元素。
```java

List<E> result = Lists.newArrayList();
PeekingIterator<E> iter = Iterators.peekingIterator(source.iterator());
while (iter.hasNext()) {
  E current = iter.next();
  while (iter.hasNext() && iter.peek().equals(current)) {
    // 跳过重复的元素
    iter.next();
  }
  result.add(current);
}
```

传统的实现方式需要记录上一个元素，并在特定情况下后退，但这很难处理且容易出错。相较而言，`PeekingIterator`是比较直接的理解和使用。

# AbstractIterator
实现你自己的 `Iterator`?  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/AbstractIterator.html'><code>AbstractIterator</code></a> 让生活更轻松。

用一个例子来解释 `AbstractIterator` 最简单。比方说，我们要包装一个 `iterator` 以便跳过空值。

```java
public static Iterator<String> skipNulls(final Iterator<String> in) {
  return new AbstractIterator<String>() {
    protected String computeNext() {
      while (in.hasNext()) {
        String s = in.next();
        if (s != null) {
          return s;
        }
      }
      return endOfData();
    }
  };
}
```

你实现了<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/AbstractIterator.html#computeNext()'><code>computeNext()</code></a>, 来计算下一个值。如果循环结束了也没有找到下一个值，请返回`endOfData()`表明已经到达迭代的末尾。

注意：`AbstractIterator` 继承了 `UnmodifiableIterator`，所以禁止实现 `remove()`方法。如果你需要支持 `remove()`的迭代器，就不应该继承 `AbstractIterator`。

## AbstractSequentialIterator
一些迭代器更容易以其他方式表达。 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/AbstractSequentialIterator.html'><code>AbstractSequentialIterator</code></a> 提供了另一种表达迭代的方法。
```java
   Iterator<Integer> powersOfTwo = new AbstractSequentialIterator<Integer>(1) { // 注意初始值1!
     protected Integer computeNext(Integer previous) {
       return (previous == 1 << 30) ? null : previous * 2;
     }
   };
```

这里，我们实现方法<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/AbstractSequentialIterator.html#computeNext(T)'><code>computeNext(T)</code></a>，它能接受前一个值作为参数。

注意，你必须额外传入一个初始值，或者传入 `null` 让迭代立即结束。因为 `computeNext(T)`假定 `null` 值意味着迭代的末尾——`AbstractSequentialIterator` 不能用来实现可能返回 `null` 的迭代器。
