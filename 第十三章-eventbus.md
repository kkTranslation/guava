`EventBus` 允许组件之间发布 - 订阅式通信，而不需要组件彼此之间显式地注册（并且因此意识到彼此）。 它专门用于使用显式注册来代替传统的Java进程内事件分发。 它不是通用型的发布-订阅实现，不适用于进程间通信。

# 范例
```java

// Class通常由容器注册。
class EventBusChangeRecorder {
  @Subscribe 
  public void recordCustomerChange(ChangeEvent e) {
    recordChange(e.getChange());
  }
}
// somewhere during initialization
eventBus.register(new EventBusChangeRecorder());
// much later
public void changeCustomer() {
  ChangeEvent event = getChangeEvent();
  eventBus.post(event);
}
```


# 一分钟指南

将现有的基于`EventListener`的系统转换为使用`EventBus`很容易。

## 事件监听者[Listeners]

监听特定事件：(如， `CustomerChangeEvent`)...

* **...在传统的Java事件中:** 定义相应的事件监听者类，如 `CustomerChangeEventListener`.
* **...`EventBus` 实现:** 以`CustomerChangeEvent` 为唯一参数创建方法，并用 [`@Subscribe`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/Subscribe.html) 注解标记。

把事件监听者注册到事件生产者：

* **...在传统的Java事件中:** 调用事件生产者的 `registerCustomerChangeEventListener` 方法；这些方法很少定义在公共接口中，因此开发者必须知道所有事件生产者的类型，才能正确地注册监听者；
* **...`EventBus`实现:** 在 `EventBus` 实例上调用 [`EventBus.register(Object)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/EventBus.html#register(java.lang.Object)) 方法；请保证事件生产者和监听者共享相同的 `EventBus`实例。

按事件超类监听 (如，`EventObject` 或 `Object`)...

* **...在传统的Java事件中:** 很困难，需要开发者自己去实现匹配逻辑；
* **...`EventBus`实现:** EventBus 自动把事件分发给事件超类的监听者，并且允许监听者声明监听接口类型和泛型的通配符类型

检测没有监听者的事件：

* **...在传统的Java事件中:** 在每个事件分发方法中添加逻辑代码（也可能适用 AOP）；
* **...`EventBus`实现:** 监听 [`DeadEvent`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/DeadEvent.html)；`EventBus` 会把所有发布后没有监听者处理的事件包装为`DeadEvent`（对调试很便利）。

## 事件生产者[Producers]
管理和追踪监听者：

* **...在传统的Java事件中:** 列表管理监听者，还要考虑线程同步；或者使用工具类，如`EventListenerList`；`EventBus`实现：EventBus 内部已经实现了监听者管理。
* **...`EventBus`实现:** `EventBus` 为你做这个。


向监听者分发事件：

* **...在传统的Java事件中:** 编写一个方法来将事件分派给每个事件监听器，包括事件类型匹配、异常处理、异步分发
* **...`EventBus`实现:** 把事件传递给 [`EventBus.post(Object)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/EventBus.html#post(java.lang.Object)) 方法。异步分发可以直接用 `EventBus` 的子类 `AsyncEventBus`。


# 术语表

`EventBus`系统和代码使用以下术语来讨论事件分发：

| 事件            | 可以向事件总线发布的对象 |
| :--------------- | :--------------------------------------- |
| 订阅      | 向事件总线注册<em>监听者</em>以接受事件的行为 |
| 监听者         | 提供一个<em>处理方法</em>，希望接受和处理事件的对象 |
| 处理方法   | 监听者提供的公共方法，事件总线使用该方法向监听者发送事件；该方法应该用[`@Subscribe`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/Subscribe.html "annotation in com.google.common.eventbus") 注解 |
| 发布消息 | 通过事件总线向所有匹配的<em>监听者</em>提供事件 |

# 常见问题解答[FAQ]
### 为什么一定要创建 EventBus 实例，而不是使用单例模式？
`EventBus`没有指定如何使用它; 你可以在应用程序中按照不同的组件、上下文或业务主题分别使用不同的事件总线。这也使得在测试中设置和拆除`EventBus`对象变得很简单。

当然，如果你想在进程范围内使用唯一的事件总线，你也可以自己这么做。比如在容器中声明 `EventBus` 为全局单例，或者用一个静态字段存放 `EventBus`，如果你喜欢的话。

简而言之，`EventBus` 不是单例模式，是因为我们不想为你做这个决定。你喜欢怎么用就怎么用吧。

### 我可以从事件总线中注销监听者吗？
是的，使用`EventBus.unregister`，但我们发现这只是很少需要：

* 大多数监听者都是在启动或者模块懒加载时注册的，并且在应用程序的整个生命周期都存在；
* 范围特定的EventBus实例可以处理临时事件分发（例如，在请求范围[request-scoped]内的对象之间分发事件）
    * 对于测试，`EventBus`实例可以很容易地创建和抛弃，消除了显式注销的需要。

### 为什么使用注解标记处理方法，而不是要求监听者实现接口？

我们觉得`@Subscribe` 注解和实现接口一样传达了明确的语义，甚至可能更好。同时，使用注解也允许你把处理方法放到任何地方，和使用业务意图清晰的方法命名。

传统的 Java 实现中，监听者使用方法很少的接口——通常只有一个方法。这样做有一些缺点:

* 监听者类对给定事件类型，只能有单一处理逻辑；
* 监听器接口方法可能会发生冲突。
    * 方法命名只和事件相关（`handleChangeEvent`），不能表达意图（例如 `recordChangeInJournal`）；
    * 事件通常有自己的接口，而没有按类型定义的公共父接口（如所有的UI事件接口）。

接口实现监听者的方式很难做到简洁，这甚至引出了一个模式，尤其是在 Swing 应用中，那就是用匿名类实现事
件监听者的接口。

比较以下两种实现：

```java

   class ChangeRecorder {
     void setCustomer(Customer cust) {
       cust.addChangeListener(new ChangeListener() {
         public void customerChanged(ChangeEvent e) {
           recordChange(e.getChange());
         }
       };
     }
   }
```
与
```java

   // 这个监听者类通常由容器注册给事件总线
   class EventBusChangeRecorder {
     @Subscribe public void recordCustomerChange(ChangeEvent e) {
       recordChange(e.getChange());
     }
   }
```

> 第二种实现的业务意图明显更加清晰：没有多余的代码，并且处理方法的名字是清晰和有意义的。

### 通用的监听者接口 `Handler<T>` 怎么样？
> 一些人为`EventBus`监听器提出了一个通用的`Handler <T>`接口。 这遇到了Java使用类型擦除的问题，更不用说可用性的问题。

假设我们有如下这个接口：
```java
interface Handler<T> {
  void handleEvent(T event);
}
```

> 由于擦除，没有单个类可以多次使用不同类型参数实现通用接口。 这是传统Java事件的一个巨大的倒退，即使`actionPerformed`和`keyPressed`不是非常有意义的名称，至少你可以实现这两种方法！

### `EventBus`不会破坏静态类型并消除自动重构支持吗？
有些人被 EventBus 的 `register(Object)` 和 `post(Object)` 方法直接使用 `Object` 做参数吓坏了。

这里使用 `Object` 参数有一个很好的理由：`EventBus` 对事件监听者类型(如在 `register(Object)`)和事件本身的类型 (在`post(Object)`)都不作任何限制。

另一方面，处理方法必须要明确地声明参数类型——期望的事件类型（或事件的父类型）。因此，搜索一个事件的类型引用，可以马上找到针对该事件的处理方法，对事件类型的重命名也会在 IDE （以及创建事件的任何代码）中自动更新所有的处理方法。

在 EventBus 的架构下，你可以任意重命名`@Subscribe` 注解的处理方法，并且这类重命名不会被传播（即不会引起其他类的修改），因为对 EventBus 来说，处理方法的名字是无关紧要的。如果测试代码中直接调用了处理方法，那么当然，重命名处理方法会引起测试代码的变动，但使用 EventBus 触发处理方法的代码就不会发生变
更。我们认为这是 EventBus 的特性，而不是漏洞：能够任意重命名处理方法，可以让你的处理方法命名更清晰。

### 如果我注册了一个没有任何处理方法的监听者，会发生什么？
什么也不会发生。

`EventBus` 旨在与容器和模块系统整合，Guice 就是个典型的例子。在这种情况下，可以方便地让容器/工厂/运行环境传递任意创建好的对象给 `EventBus` 的 `register(Object)`方法。

这样，任何容器/工厂/运行环境创建的对象都可以简便地通过暴露处理方法挂载到系统的事件模块。
### 编译时能检测到 EventBus 的哪些问题？
Java 类型系统可以明白地检测到的任何问题。比如，为一个不存在的事件类型定义处理方法。

### 运行时往 EventBus 注册监听者，可以立即检测到哪些问题？
一旦调用了 `register(Object)` 方法，`EventBus` 就会检查监听者中的处理方法是否结构正确的[well-formednes
s]。具体来说，就是每个用`@Subscribe` 注解的方法都只能有一个参数。

违反这条规则将引起 `IllegalArgumentException`（这条规则检测也可以用 APT 在编译时完成，不过我们还在研究中）。

### 哪些问题只能在之后事件传播的运行时才会被检测到？
如果组件传播了一个事件，但找不到相应的处理方法，`EventBus` 可能会指出一个错误（通常是指出`@Subscribe`注解的缺失，或没有加载监听者组件）。
请注意这个指示并不一定表示应用有问题。一个应用中可能有好多场景会故意忽略某个事件，尤其当事件来源于不可控代码时

你可以注册一个处理方法专门处理 DeadEvent 类型的事件。每当 EventBus 收到没有对应处理方法的事件，它都会将其转化为 DeadEvent，并且传递给你注册的 DeadEvent 处理方法——你可以选择记录或修复该事件。

### 怎么测试监听者和它们的处理方法？
因为监听者的处理方法都是普通方法，你可以简便地在测试代码中模拟 `EventBus` 调用这些方法。

## 为什么我不能在 EventBus 上使用<泛型魔法>？
`EventBus`旨在很好地处理一大类用例。我们更喜欢针对大多数用例直击要害，而不是在所有用例上都保持体
面。

此外，泛型也让 `EventBus` 的可扩展性——让它有益、高效地扩展，同时我们对 `EventBus` 的增补不会和你们的扩展相冲突——成为一个非常棘手的问题。

如果你真的很想用泛型，`EventBus` 目前还不能提供，你可以提交一个问题并且设计自己的替代方案。
