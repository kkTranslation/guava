# 概述
Guava <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html'><code>Service</code></a>接口表示具有操作状态的对象，具有start和stop的方法。 例如，Web服务器，RPC服务器和计时器可以实现`Service`接口。 管理这些需要适当启动和关闭管理的服务的状态可以是不重要的，特别是如果涉及多线程或调度时。 Guava提供了一些框架来管理状态逻辑和同步细节。

# 使用一个服务

`Service`的正常生命周期是

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>Service.State.NEW</code></a> to
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STARTING'><code>Service.State.STARTING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#RUNNING'><code>Service.State.RUNNING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STOPPING'><code>Service.State.STOPPING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#TERMINATED'><code>Service.State.TERMINATED</code></a>

停止的服务可能无法重新启动。 如果服务在启动，运行或停止时失败，它将进入状态<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#FAILED'><code>Service.State.FAILED</code></a>。

可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>startAsync()</code></a>异步启动服务，它返回此值以启用方法链。 它只有在服务为<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>NEW</code></a>时调用`startAsync()`才有效。 因此，你应该将应用程序结构化，以便在每个服务启动时都有一个唯一的位置。 

停止服务类似，使用异步<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#stopAsync()'><code>stopAsync()</code></a> 方法。 但不同于<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>startAsync()</code></a>，可以安全地多次调用此方法。 这是为了可以处理在关闭服务时可能发生的竞赛。

Service还提供了几种方法来等待服务转换完成。
* 异步的使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#addListener()'><code>addListener()</code></a>。`addListener()`允许你添加将在服务的每个状态转换上调用的Service.Listener <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.Listener.html'><code>Service.Listener</code></a>。 N.B.如果在添加侦听器时服务不是<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>NEW</code></a>，则已经发生的任何状态转换将不会在侦听器上重播。
* 同步使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitRunning()'><code>awaitRunning()</code></a>。 这是不可中断的，不抛出异常，并在服务完成启动后返回。 如果服务无法启动，则会抛出`IllegalStateException`。 类似地，<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitTerminated()'><code>awaitTerminated()</code></a>等待服务到达终端状态(<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#TERMINATED'><code>TERMINATED</code></a>或<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#FAILED'><code>FAILED</code></a>)。 这两种方法都有重载，允许指定超时。 

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html'><code>Service</code></a> 接口是微妙和复杂的。 我们不建议直接实现它。 相反，请使用guava中的一个抽象基类作为你的实现的基础。 每个基类支持特定的线程模型。

# Implementations
## AbstractIdleService
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html'><code>AbstractIdleService</code></a> 框架实现一个Service，它不需要在“running”状态下执行任何操作，因此在运行时不需要线程 - 但是具有要执行的startup和shutdown操作。 实现这样的服务就像扩展`AbstractIdleService`和实现<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#startUp()'><code>startUp()</code></a> 和 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#shutDown()'><code>shutDown()</code></a>方法一样简单。 

```java
protected void startUp() {
  servlets.add(new GcStatsServlet());
}
protected void shutDown() {}
```

注意，对`GcStatsServlet`的任何查询都已经有一个线程运行。我们不需要这个服务在服务运行时自己执行任何操作。

## AbstractExecutionThreadService
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html'><code>AbstractExecutionThreadService</code></a> 在单个线程中执行启动，运行和关闭操作。 你必须覆盖<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html#run()'><code>run()</code></a> 方法，并且它必须响应停止请求。 例如，你可以在工作循环中执行操作：

```java
public void run() {
  while (isRunning()) {
    // perform a unit of work
  }
}
```

或者，你可以以任何导致`run()`返回的方式覆盖<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html#triggerShutdown()'><code>triggerShutdown()</code></a>。

覆盖`startUp()`和`shutDown()`是可选的，但将为你管理服务状态。

```java
protected void startUp() {
  dispatcher.listenForConnections(port, queue);
}
protected void run() {
  Connection connection;
  while ((connection = queue.take() != POISON)) {
    process(connection);
  }
}
protected void triggerShutdown() {
  dispatcher.stopListeningForConnections(queue);
  queue.put(POISON);
}
```

注意，`start()`调用你的`startUp()`方法，为你创建一个线程，并在该线程中调用`run()`。 `stop()`调用`triggerShutdown()`并等待线程死亡。

## AbstractScheduledService
<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html'><code>AbstractScheduledService</code></a> 在运行时执行一些周期性任务。 子类实现<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#runOneIteration()'><code>runOneIteration()</code></a> 以遍历一个指定任务，以及熟悉的startUp和`shutDown()`方法。

要描述执行计划，必须实现<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#scheduler()'><code>scheduler()</code></a> 方法。 通常，你将使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html'><code>AbstractScheduledService.Scheduler</code></a>提供的计划之一，即<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedRateSchedule(long, long, java.util.concurrent.TimeUnit)'><code>newFixedRateSchedule(initialDelay, delay, TimeUnit)</code></a> 或 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedDelaySchedule(long, long, java.util.concurrent.TimeUnit)'><code>newFixedDelaySchedule(initialDelay, delay, TimeUnit)</code></a>，对应于`ScheduledExecutorService`中熟悉的方法。 自定义计划可以使用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.CustomScheduler.html'><code>CustomScheduler</code></a>实现; 有关详细信息，请参阅Javadoc。

## AbstractService
当你需要做自己的手工线程管理时，直接覆盖<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html'><code>AbstractService</code></a> 。 通常，你应该很好地服务于上面的实现，但是推荐实现`AbstractService`，例如，当你正在建模一些提供自己的线程语义作为一个`Service`，你有自己的特定线程需求。 

要实现`AbstractService`，你必须实现2种方法。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStart()'><code>doStart()</code></a>: ` doStart()`被第一次调用`startAsync()`直接调用，你的`doStart()`方法应该执行所有初始化，然后最终调用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStarted()'><code>notifyStarted()</code></a> 如果启动成功或<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed(java.lang.Throwable)'><code>notifyFailed()</code></a> 如果启动失败。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStop()'><code>doStop()</code></a>:  `doStop()` 被第一次调用`stopAsync()`直接调用，你的`doStop()`方法应该关闭你的服务，然后最终调用<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStopped()'><code>notifyStopped()</code></a> 如果关闭成功或<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed(java.lang.Throwable)'><code>notifyFailed()</code></a> 如果关闭失败。

你的`doStar`t和`doStop`，方法应该快。 如果你需要做昂贵的初始化，例如读取文件，打开网络连接或任何可能阻塞的操作，你应该考虑将该工作移动到另一个线程。

# 使用ServiceManager

除了服务框架实现，Guava提供了一个<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html'><code>ServiceManager</code></a> 类，使得涉及多个服务实现的某些操作更容易。 使用服务集合创建一个新的`ServiceManager`。 然后你可以管理他们：

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startAsync()'><code>startAsync()</code></a> 将启动所有正在管理的服务。 很像<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>Service#startAsync()</code></a> ，如果所有的服务都是`NEW`，你只能调用这个方法一次。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#stopAsync()'><code>stopAsync()</code></a> 将停止所有正在管理的服务。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#addListener(com.google.common.util.concurrent.ServiceManager.Listener, java.util.concurrent.Executor)'><code>addListener</code></a>将添加一个<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.Listener.html'><code>ServiceManager.Listener</code></a> ，它将在主要状态转换时被调用。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitHealthy()'><code>awaitHealthy()</code></a> 将等待所有服务达到`RUNNING`状态。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitStopped()'><code>awaitStopped()</code></a> 将等待所有服务达到终端状态。

或检查他们：
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#isHealthy()'><code>isHealthy()</code></a>如果所有服务都为`RUNNING`，则返回`true`。
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#servicesByState()'><code>servicesByState()</code></a> 返回由其状态索引的所有服务的一致性快照。
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startupTimes()'><code>startupTimes()</code></a> 将从管理下的`Service`返回一个map到该服务以毫秒为单位启动所需的时间。 返回的map保证按启动时间排序。

虽然建议通过`ServiceManager`管理服务生命周期，但通过其他机制发起的状态转换**不会影响其方法的正确性**。 例如，如果服务由`startAsync()`之外的某种机制启动，则监听器将在适当时被调用，并且`awaitHealthy()`仍将按预期工作。 `ServiceManager`强制执行的唯一要求是，在构建`ServiceManager`时，所有服务都必须为新。
