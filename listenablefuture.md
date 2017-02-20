并发编程是一个难题，但是一个强大而简单的抽象可以显著的简化并发的编写。出于这样的考虑，Guava 定义了
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html'><code>ListenableFuture</code></a>接口并继承了 JDK concurrent 包下的 `Future `接口。

**我们强烈建议您在所有代码中始终使用ListenableFuture而不是Future，**, 因为：
* 大多数 `Futures` 方法中需要它。
* 这比以后改用`ListenableFuture`更容易。
    * Guava 提供的通用公共类封装了公共的操作方方法，不需要提供`Future` 和`ListenableFuture`的扩展方法。

# 接口
传统 JDK 中的 `Future` 通过异步的方式计算返回结果:在多线程运算中可能或者可能在没有结束返回结果，`Future` 是运行中的多线程的一个引用句柄，确保在服务执行返回一个 Result。

`ListenableFuture`可以允许你注册回调方法(callbacks)，在运算（多线程执行）完成的时候进行调用, 或者在运算（多线程执行）完成后立即执行。这样简单的改进，使得可以明显的支持更多的操作，这样的功能在 JDK con
current 中的 `Future` 是不支持的。

`ListenableFuture` 中的基础方法是 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html#addListener(java.lang.Runnable, java.util.concurrent.Executor)'><code>addListener(Runnable, Executor)</code></a>，该方法会在多线程运算完的时候，指定的 Runnable 参数传入的对象会被指定的 `Executor` 执行。

# 添加回调（Callbacks）
大多数用户更喜欢使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback, java.util.concurrent.Executor)'><code>Futures.addCallback(ListenableFuture&lt;V&gt;, FutureCallback&lt;V&gt;, Executor)</code></a>, 或默认使用`MoreExecutors.sameThreadExecutor()`的 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback)'>version</a> ，以便采用轻量级的设计。<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html'><code>FutureCallback&lt;V&gt;</code></a> 实现两种方法：
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onSuccess(V)'><code>onSuccess(V)</code></a>, 在 Future 成功的时候执行，根据 Future 结果来判断。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onFailure(java.lang.Throwable)'><code>onFailure(Throwable)</code></a>, 在 Future 失败的时候执行，根据 Future 结果来判断。
# ListenableFuture 的创建

对应 JDK 中的 <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/ExecutorService.html#submit(java.util.concurrent.Callable)'><code>ExecutorService.submit(Callable)</code></a> 提交多线程异步运算的方式，Guava 提供了 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListeningExecutorService.html'><code>ListeningExecutorService</code></a> 接口, 该接口返回 `ListenableFuture` 而相应的`ExecutorService`返回普通的`Future`。将 `ExecutorService`转为`ListeningExecutorService`，可以使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/MoreExecutors.html#listeningDecorator(java.util.concurrent.ExecutorService)'><code>MoreExecutors.listeningDecorator(ExecutorService)</code></a>进行装饰。

```java
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
ListenableFuture<Explosion> explosion = service.submit(new Callable<Explosion>() {
  public Explosion call() {
    return pushBigRedButton();
  }
});
Futures.addCallback(explosion, new FutureCallback<Explosion>() {
  // we want this handler to run immediately after we push the big red button!
  public void onSuccess(Explosion explosion) {
    walkAwayFrom(explosion);
  }
  public void onFailure(Throwable thrown) {
    battleArchNemesis(); // escaped the explosion!
  }
});
```

另外, 假如你是从  <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/FutureTask.html'><code>FutureTask</code></a>转换而来的, Guava 提供<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.util.concurrent.Callable)'><code>ListenableFutureTask.create(Callable&lt;V&gt;)</code></a> 和 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.lang.Runnable, V)'><code>ListenableFutureTask.create(Runnable, V)</code></a>.和 JDK 不同的是,`ListenableFutureTask`不能随意被继承（译者注：ListenableFutureTask 中的 done 方法实现了调用 listener 的操作）。

假如你喜欢抽象的方式来设置 future 的值，而不是想实现接口中的方法，可以考虑继承抽象类 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractFuture.html'><code>AbstractFuture&lt;V&gt;</code></a> 或者直接使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/SettableFuture.html'><code>SettableFuture</code></a> 。

假如你必须将其他 API 提供的 `Future` 转换成 `ListenableFuture`，你没有别的方法只能采用硬编码的方式 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/JdkFutureAdapters.html'><code>JdkFutureAdapters.listenInPoolThread(Future)</code></a> 来将 `Future` 转换成 `ListenableFuture`。尽可能地采用修改原生
的代码返回 ListenableFuture 会更好一些。

# Application
使用 `ListenableFuture` 最重要的理由是它可以进行一系列的复杂链式的异步操作。

```java
ListenableFuture<RowKey> rowKeyFuture = indexService.lookUp(query);
AsyncFunction<RowKey, QueryResult> queryFunction =
  new AsyncFunction<RowKey, QueryResult>() {
    public ListenableFuture<QueryResult> apply(RowKey rowKey) {
      return dataService.read(rowKey);
    }
  };
ListenableFuture<QueryResult> queryFuture =
    Futures.transformAsync(rowKeyFuture, queryFunction, queryExecutor);
```

其他更多的操作可以有效地支持`ListenableFuture`，不能单独支持`Future`。 不同的操作可以由不同的执行器执行，并且单独的 `ListenableFuture`可以有多个操作等待。

当一个操作开始的时候其他的一些操作也会尽快开始执行–“fan-out”–`ListenableFuture` 能够满足这样的场景：促发所有的回调（callbacks）。反之更简单的工作是，同样可以满足“fan-in”场景，促发 ListenableFuture 获取（get）计算结果，同时其它的 Futures 也会尽快执行：可以参考 <a href='http://google.github.io/guava/releases/snapshot/api/docs/src-html/com/google/common/util/concurrent/Futures.html#line.1276'>the implementation of <code>Futures.allAsList</code></a>。

| 方法                                   | 描述                              | 参考                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction, java.util.concurrent.Executor)'><code>transformAsync(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;, Executor)</code></a>`*` | 返回一个新的`ListenableFuture`，返回的 result 是由传入的 AsyncFunction 参数指派到传入的`ListenableFuture`的结果的乘积。 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)'><code>transformAsync(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function, java.util.concurrent.Executor)'><code>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;, Executor)</code></a> | 返回一个新的`ListenableFuture`，返回的 result 是将给定的`Function`应用于给定`ListenableFuture`的结果的乘积。 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)'><code>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList(java.lang.Iterable)'><code>allAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</code></a> | 返回一个`ListenableFuture`，它的值是一个包含每个输入futures的值的列表。 如果任何输入futures 失败或被取消，此future失败或被取消。 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList(com.google.common.util.concurrent.ListenableFuture...)'><code>allAsList(ListenableFuture&lt;V&gt;...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList(java.lang.Iterable)'><code>successfulAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</code></a> | 返回一个`ListenableFuture'，其值是一个包含每个成功输入Futures的值的列表。 与失败或取消的future相对应的值将替换为null。 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList(com.google.common.util.concurrent.ListenableFuture...)'><code>successfulAsList(ListenableFuture&lt;V&gt;...)</code></a> |

`*` <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AsyncFunction.html'><code>AsyncFunction&lt;A, B&gt;</code></a> 提供一种方法，ListenableFuture<B> apply(A input)。它可以用于异步变换值。

```java
List<ListenableFuture<QueryResult>> queries;
// The queries go to all different data centers, but we want to wait until they're all done or failed.

ListenableFuture<List<QueryResult>> successfulQueries = Futures.successfulAsList(queries);

Futures.addCallback(successfulQueries, callbackOnSuccessfulQueries);
```

# CheckedFuture
Guava还提供了一个 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/CheckedFuture.html'><code>CheckedFuture&lt;V, X extends Exception&gt;</code></a> 接口。 `CheckedFuture`是一个`ListenableFuture`，它包含可以抛出已检查异常的get方法的版本。 这使得更容易创建一个执行可以抛出异常的逻辑的`future`。 要将`ListenableFuture`转换为`CheckedFuture`，请使用 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#makeChecked(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)'><code>Futures.makeChecked(ListenableFuture&lt;V&gt;, Function&lt;Exception, X&gt;)</code></a>。
