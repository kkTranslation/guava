# Example

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .expireAfterWrite(10, TimeUnit.MINUTES)
       .removalListener(MY_LISTENER)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
```

# Applicability

Caches在各种用例中非常有用。 例如，当一个值计算或检索成本高昂时，您应该考虑使用高速缓存，并且您需要在某个输入上多次使用它的值。

`Cache`类似于`ConcurrentMap`，但不完全相同。 最根本的区别是，ConcurrentMap会持续添加到其中的所有元素，直到它们被显式删除。 另一方面，`Cache`通常被配置为自动驱逐条目，以限制其内存占用。 在某些情况下，由于其自动缓存加载，所以`LoadCache`可能很有用。

通常，Guava缓存实用程序适用于以下情况：

* 你愿意花一些时间来提高速度。
* 你期望keys有时会被多次查询。
    * 您的缓存不需要存储比RAM内容更多的数据。 （Guava缓存是本地应用程序的单一运行，它们不会将数据存储在文件或外部服务器上，如果这不符合您的需要，请考虑使用[Memcached](http://memcached.org/)等工具。）

如果这些都适用于您的用例，那么Guava缓存实用程序可能适合您！

获取`Cache`是使用上面的示例代码所示的`CacheBuilder`构建器模式完成的，但是自定义缓存是有趣的部分。

_注意:_ ：如果不需要`Cache`的功能，`ConcurrentHashMap`更具有内存效率 - 但是使用任何旧的`ConcurrentMap`来复制大多数`Cache`功能是非常困难或不可能的。

# Population

问自己关于缓存的第一个问题是：是否有一些_sensible default_函数来加载或计算与一个键相关联的值？ 如果是这样，你应该使用一个`CacheLoader`。 如果没有，或者您需要覆盖默认值，但您仍然希望使用原子`get-if-absent-compute`语义，则应将`Callable`传递给`get`调用。 可以使用`Cache.put`直接插入元素，但是首选自动缓存加载，因为它可以更容易地理解所有缓存内容的一致性。

### From a [CacheLoader](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html)

`LoadCache`是使用附加的`CacheLoader`构建的`Cache`。 创建`CacheLoader`通常与实现方法`V load(K key) throws Exception`一样简单。 所以，例如，您可以使用以下代码创建一个`LoadCache`：

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });

...
try {
  return graphs.get(key);
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

查询`LoadCache`的规范方法是使用[get(K)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#get(K)).方法。 这将返回已经缓存的值，否则使用缓存的`CacheLoader`将新值原子加载到缓存中。 因为`CacheLoader`可能会抛出`Exception`，所以`LoadingCache.get(K)`抛出`ExecutionException`。 如果您定义了一个不声明任何检查异常的`CacheLoader`，那么可以使用`getUnchecked(K)`执行缓存查找; 但是请注意不要在`CacheLoader`声明检查异常的缓存上调用`getUnchecked`。

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .expireAfterAccess(10, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });

...
return graphs.getUnchecked(key);
```

可以使用`getAll(Iterable<? extends K>)`方法执行批量查找。 默认情况下，`getAll`将为高速缓存中缺少的每个键单独调用`CacheLoader.load`。 当批量检索比许多单独的查找更有效时，您可以覆盖<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#loadAll(java.lang.Iterable)'><code>CacheLoader.loadAll</code></a>来利用这一点。 `getAll(Iterable)`的性能将相应提高。

请注意，您可以编写一个`CacheLoader.loadAll`实现，该实现加载未特别请求的键的值。 例如，如果从某个组计算任何key的值，您将获得组中所有keys的值，`loadAll`可能会同时加载该组的其余部分。

### From a <a href='http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html'><code>Callable</code></a>

所有Guava缓存，加载或不加载，支持方法<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#get(java.lang.Object,java.util.concurrent.Callable)'><code>get(K, Callable&lt;V&gt;)</code></a>。 此方法返回与缓存中的key相关联的值，或者从指定的`Callable`计算它，并将其添加到缓存。 在加载完成之前，修改与此缓存关联的可观察状态。 该方法提供了传统的"if cached, return; otherwise create, cache and return"模式的简单替代。

```java

Cache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .build(); // look Ma, no CacheLoader
...
try {
  // If the key wasn't in the "easy to compute" group, we need to
  // do things the hard way.
  cache.get(key, new Callable<Value>() {
    @Override
    public Value call() throws AnyException {
      return doThingsTheHardWay(key);
    }
  });
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

### Inserted Directly

Values 可以直接用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#put(K, V)'><code>cache.put(key, value)</code></a>插入到缓存中。 这将覆盖指定key缓存中的任何先前entry。 也可以使用`Cache.asMap()`视图公开的任何`ConcurrentMap`方法对缓存进行更改。 请注意，`asMap`视图中的任何方法都不会导致entry自动加载到缓存中。 此外，该视图上的原子操作超出了自动缓存加载的范围，因此`Cache.get(K, Callable<V>)`应始终优于`Cache.asMap().putIfAbsent`在使用`CacheLoader`或 `Callable`。

# Eviction

冷酷的现实是，我们几乎确实没有足够的内存来缓存我们可以缓存的所有东西。 你必须决定：什么时候不值得保存缓存entry？ Guava提供三种基本类型的eviction：基于size-based eviction, time-based eviction,和基于reference-based eviction。

## Size-based Eviction

如果您的缓存不应超过一定大小，请使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumSize(long)'><code>CacheBuilder.maximumSize(long)</code></a>。 缓存将尝试驱逐最近或经常未被使用的entries。 警告：超出此限制之前，缓存可能会超出entries - 通常当缓存大小接近极限时。

或者，如果不同的缓存entries具有不同的 "weights" - 例如，如果您的缓存值具有完全不同的内存占用空间，则可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weigher(com.google.common.cache.Weigher)'><code>CacheBuilder.weigher(Weigher)</code></a>指定权重函数，并使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumWeight(long)'><code>CacheBuilder.maximumWeight(long)</code></a>。 除了与`maxSize`相同的注意事项外，请注意，weights是在entry创建时计算的，此后将是静态的。

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumWeight(100000)
       .weigher(new Weigher<Key, Graph>() {
          public int weigh(Key k, Graph g) {
            return g.vertices().size();
          }
        })
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });
```

## Timed Eviction

`CacheBuilder`提供了两种定时eviction的方法：

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterAccess(long, java.util.concurrent.TimeUnit)'><code>expireAfterAccess(long, TimeUnit)</code></a> 只有在从读取或写入上次访问该entry 以来，指定的持续时间已过去的entry 才会过期。 请注意，entry被逐出的顺序将类似于 [size-based eviction](#Size-based-Eviction).
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterWrite(long, java.util.concurrent.TimeUnit)'><code>expireAfterWrite(long, TimeUnit)</code></a> 在创建entry后指定的持续时间过后或最近替换该值的过期entry。 如果缓存的数据在一段时间后变长，这可能是可取的。

Timed expiration 是在写入期间进行定期维护，偶尔在读取期间执行，如下所述。

### Testing Timed Eviction

测试timed eviction并不一定是痛苦的......而实际上并没有把你两秒测试两秒到期。 使用[Ticker](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Ticker.html) 接口和<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#ticker(com.google.common.base.Ticker)'><code>CacheBuilder.ticker(Ticker)</code></a> 方法在缓存生成器中指定时间源，而不必等待系统时钟。

## Reference-based Eviction

Guava可以让你设置你的缓存，让entries的垃圾收集，通过使用键或值，[弱引用](http://docs.oracle.com/javase/6/docs/api/java/lang/ref/WeakReference.html) 和使用值的 [软引用](http://docs.oracle.com/javase/6/docs/api/java/lang/ref/SoftReference.html) 。

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakKeys()'><code>CacheBuilder.weakKeys()</code></a> 存储使用弱引用的键。 如果没有其他（强或软）对键的引用，则允许entries被垃圾回收。 由于垃圾收集仅取决于身份相等性，所以这导致整个缓存使用identity（`==`）相等来比较keys而不是 `equals()`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakValues()'><code>CacheBuilder.weakValues()</code></a> 使用弱引用存储值。 如果没有其他（强或软）对值的引用，则允许条目被垃圾回收。 由于垃圾收集仅取决于身份相等性，所以这导致整个缓存使用identity（`==`）相等来比较值而不是`equals()`.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#softValues()'><code>CacheBuilder.softValues()</code></a> 在软引用中包装值。 响应于内存需求，软件引用的对象以全局最不近期使用的方式进行垃圾回收。 由于使用软引用的性能影响，我们通常建议使用更可预测的最大缓存大小。 使用`softValues()`将导致使用identity（`==`）相等而不是`equals()`来比较值。  

## 明确删除

在任何时候，你可以明确地无效缓存entries，而不是等待被驱逐entries。 这可以做到：

* 单独使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidate(java.lang.Object)'><code>Cache.invalidate(key)</code></a>
* 大量使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll(java.lang.Iterable)'><code>Cache.invalidateAll(keys)</code></a>
    * 对所有entries，使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll()'><code>Cache.invalidateAll()</code></a>

## 移除 Listeners

您可以通过 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#removalListener(com.google.common.cache.RemovalListener)'><code>CacheBuilder.removalListener(RemovalListener)</code></a>为缓存指定删除监听器，以便在删除entry时执行某些操作。 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListener.html'><code>RemovalListener</code></a> 通过一个 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalNotification.html'><code>RemovalNotification</code></a>，它指定了 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalCause.html'><code>RemovalCause</code></a>, key 和 value。

请注意，`RemovalListener`抛出的任何异常都会被记录（使用`Logger`）并被吞噬。

```java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
  public DatabaseConnection load(Key key) throws Exception {
    return openConnection(key);
  }
};
RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
  public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
    DatabaseConnection conn = removal.getValue();
    conn.close(); // tear down properly
  }
};

return CacheBuilder.newBuilder()
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

警告：默认情况下同步执行删除侦听器操作，并且由于缓存维护通常在正常缓存操作期间执行，昂贵的删除监听器可能会降低正常缓存功能！ 如果您有一个昂贵的删除监听器，请使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListeners.html#asynchronous(com.google.common.cache.RemovalListener, java.util.concurrent.Executor)'><code>RemovalListeners.asynchronous(RemovalListener, Executor)</code></a> 来装饰`RemovalListener`以异步运行。

## When Does Cleanup Happen?

使用`CacheBuilder`构建的缓存不会自动执行清除和排除值，或者在值过期后立即执行或任何排序。  相反，它在写操作期间执行少量维护，或在偶尔的读操作，如果写是罕见的。

原因如下：如果我们想持续执行`Cache`维护，我们需要创建一个线程，并且它的操作将与用户操作共享锁竞争。 另外，一些环境限制线程的创建，这将使`CacheBuilder`在该环境中不可用。

相反，我们把选择放在你手中。 如果你的缓存是高吞吐量，那么你不必担心执行高速缓存维护，清理过期的entries等。 如果您的缓存只写很少，并且不希望清除来阻止缓存读取，那么您可能希望创建自己的维护线程，定期调用 <a href='http://google.github.io/guava/releases/11.0.1/api/docs/com/google/common/cache/Cache.html#cleanUp()'><code>Cache.cleanUp()</code></a> 。

如果要为只有很少写入的缓存安排常规缓存维护，只需使用 <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/ScheduledExecutorService.html'><code>ScheduledExecutorService</code></a>调度维护。

## 刷新

Refreshing 与eviction不一样。 如在 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#refresh(K)'><code>LoadingCache.refresh(K)</code></a>, 中指定的，刷新键可能会异步加载键的新值。 在刷新key时仍旧返回旧值（如果有），与eviction相反，强制检索等待，直到重新加载该值。

如果在刷新时抛出异常，则会保留旧值，并记录并吞入异常。

`CacheLoader` 可以通过覆盖 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#reload(K, V)'><code>CacheLoader.reload(K, V)</code></a>来指定刷新时使用的智能行为，从而允许您在计算新值时使用旧值。

```java
// 一些键不需要刷新，我们希望刷新可以异步完成。
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .refreshAfterWrite(1, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return getGraphFromDatabase(key);
             }

             public ListenableFuture<Graph> reload(final Key key, Graph prevGraph) {
               if (neverNeedsRefresh(key)) {
                 return Futures.immediateFuture(prevGraph);
               } else {
                 // asynchronous!
                 ListenableFutureTask<Graph> task = ListenableFutureTask.create(new Callable<Graph>() {
                   public Graph call() {
                     return getGraphFromDatabase(key);
                   }
                 });
                 executor.execute(task);
                 return task;
               }
             }
           });
```

可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#refreshAfterWrite(long, java.util.concurrent.TimeUnit)'><code>CacheBuilder.refreshAfterWrite(long, TimeUnit)</code></a>将自动定时刷新添加到缓存中。 与`expireAfterWrite`相反，`refreshAfterWrite`将在指定的持续时间后使一个有资格刷新的key，但刷新将仅在查询entry时才实际启动。 （如果`CacheLoader.reload`被实现为异步，那么查询将不会被刷新减慢）。所以，例如，您可以在同一缓存上同时指定`refreshAfterWrite`和`expireAfterWrite`，以使entry的到期定时器为 只要entry有资格刷新，就会盲目重置，所以如果entry符合刷新条件后没有查询，则允许该entry过期。

# Features

## Statistics

通过使用 <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/cache/CacheBuilder.html#recordStats()'><code>CacheBuilder.recordStats()</code></a>您可以打开Guava缓存的统计信息收集。<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#stats()'><code>Cache.stats()</code></a> 方法返回一个 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html'><code>CacheStats</code></a> 对象，它提供诸如

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#hitRate()'><code>hitRate()</code></a>, 返回命中与请求的比例
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#averageLoadPenalty()'><code>averageLoadPenalty()</code></a>, 以纳秒为单位的平均加载新值的时间
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#evictionCount()'><code>evictionCount()</code></a>, 缓存驱逐的次数

还有更多的统计数字。 这些统计数据对于缓存调整至关重要，我们建议您在关键性能的应用程序中关注这些统计信息。

## `asMap`
您可以使用其`asMap`视图将任何`Cache`视为`ConcurrentMap`，但是`asMap`视图与`Cache`的交互方式需要一些解释。

* `cache.asMap()`包含当前加载到缓存中的所有entries 。 因此，例如，`cache.asMap().keySet()`包含当前加载的所有键。
* `asMap().get(key)`本质上等同于`cache.getIfPresent(key)`，并且不会导致加载值。 这与Map契约一致。
    * 访问时间由所有缓存读写操作（包括`Cache.asMap().get(Object)`和`Cache.asMap().put(K, V)`）重置，而不是由`containsKey(Object)`, `Cache.asMap()`的集合视图。 因此，例如，通过`cache.entrySet()`迭代不会为您检索的entries重置访问时间。

# Interruption

加载方法（如`get`）不会抛出`InterruptedException`。 我们可以设计这些方法来支持`InterruptedException`，但是我们的支持是不完整的，迫使所有用户的成本，但是仅仅是一些用户的收益。 有关详细信息，请阅读。

`get` 请求未缓存值的调用分为两大类：加载值和等待另一个线程正在进行加载的值。 两者在支持中断方面有所不同。 简单的情况是等待另一个线程正在进行的加载：这里我们可以输入可中断的等待。 困难的情况是自己加载value。在这里，我们正处在一个用户提供的`CacheLoader`的帮助。 如果它支持中断，我们可以支持中断; 如果没有，我们不能。

那么为什么不提供`CacheLoader`支持中断呢？ 在某种意义上，我们做（但是请看下面）：如果`CacheLoader`抛出`InterruptedException`，那么所有获取key的调用将立即返回（就像任何其他异常一样）。 此外，`get`将恢复加载线程中的中断位。 令人惊讶的部分是`InterruptedException`被包裹在`ExecutionException`中。

原则上，我们可以为您解开这个exception。 但是，这迫使所有的`LoadCache`用户处理`InterruptedException`，即使大多数`CacheLoader`实现也不会抛出它。 也许这还是值得的，当你认为所有非加载线程的等待仍然可以中断。 但是，许多高速缓存仅在单个线程中使用。 他们的用户仍然必须抓住不可能的`InterruptedException`。 即使那些跨线程共享缓存的用户也只能有时候中断他们的`get`调用，基于哪个线程首先发出请求。

我们在这个决定中的指导原则是缓存的行为就好像所有的值被加载在调用线程中一样。 这个原则使得将缓存引入到先前在每次调用中重新计算其值的代码变得容易。 而且如果旧的代码不可中断，那么新的代码也不行。

我说在某种意义上我们支持中断。 还有另一种感觉，我们没有，使得`LoadCache`泄漏抽象。 如果加载线程中断，我们就像任何其他异常一样对待它。 在许多情况下，这很好，但是当多个呼叫正在等待该值时，这不是正确的。 虽然正在计算该值的操作已中断，但其他需要该值的操作可能并不是这样。 然而，所有这些调用者都会收到`InterruptedException`（包含在`ExecutionException`中），即使负载没有太多“fail”为“abort”。 正确的行为将是剩下的一个线程重试加载。 我们[为此提交了一个错误](https://github.com/google/guava/issues/1122)。但是，修复可能是危险的。 而不是解决问题，我们可能会在提出的`AsyncLoadingCache`中加入更多的努力，这将返回`Future`对象具有正确的中断行为。
