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
* **...`EventBus`实现:** 在 `EventBus` 实例上调用 [`EventBus.register(Object)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/EventBus.html#register(java.lang.Object) 方法；请保证事件生产者和监听者共享相同的 `EventBus`实例。

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
* **...`EventBus`实现:** 把事件传递给 [`EventBus.post(Object)`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/EventBus.html#post(java.lang.Object) 方法。异步分发可以直接用 `EventBus` 的子类 `AsyncEventBus`。


# Glossary

The `EventBus` system and code use the following terms to discuss event distribution:

| Event            | Any object that may be <em>posted</em> to a bus. |
| :--------------- | :--------------------------------------- |
| Subscribing      | The act of registering a <em>listener</em> with an `EventBus`, so that its <em>handler methods</em> will receive events. |
| Listener         | An object that wishes to receive events, by exposing <em>handler methods</em>. |
| Handler method   | A public method that the `EventBus` should use to deliver <em>posted</em> events.  Handler methods are marked by the [`@Subscribe`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/eventbus/Subscribe.html "annotation in com.google.common.eventbus") annotation. |
| Posting an event | Making the event available to any <em>listeners</em> through the `EventBus`. |

# FAQ
### Why must I create my own Event Bus, rather than using a singleton?
`EventBus` doesn't specify how you use it; there's nothing stopping your application from having separate `EventBus` instances for each component, or using separate instances to separate events by context or topic.  This also makes it trivial to set up and tear down `EventBus` objects in your tests.

Of course, if you'd like to have a process-wide `EventBus` singleton, there's nothing stopping you from doing it that way.  Simply have your container (such as Guice) create the `EventBus` as a singleton at global scope (or stash it in a static field, if you're into that sort of thing).

In short, `EventBus` is not a singleton because we'd rather not make that decision for you.  Use it how you like.

### Can I unregister a listener from the Event Bus?
Yes, using `EventBus.unregister`, but we find this is needed only rarely:

* Most listeners are registered on startup or lazy initialization, and persist for the life of the application.
* Scope-specific `EventBus` instances can handle temporary event distribution (e.g. distributing events among request-scoped objects)
    * For testing, `EventBus` instances can be easily created and thrown away, removing the need for explicit unregistration.

### Why use an annotation to mark handler methods, rather than requiring the listener to implement an interface?

We feel that the Event Bus's `@Subscribe` annotation conveys your intentions just as explicitly as implementing an interface (or perhaps more so), while leaving you free to place event handler methods wherever you wish and give them intention-revealing names.

Traditional Java Events use a listener interface which typically sports only a handful of methods -- typically one.  This has a number of disadvantages:

* Any one class can only implement a single response to a given event.
* Listener interface methods may conflict.
    * The method must be named after the event (e.g. `handleChangeEvent`), rather than its purpose (e.g. `recordChangeInJournal`).
    * Each event usually has its own interface, without a common parent interface for a family of events (e.g. all UI events).

The difficulties in implementing this cleanly has given rise to a pattern, particularly common in Swing apps, of using tiny anonymous classes to implement event listener interfaces.

Compare these two cases:
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
versus
```java

   // Class is typically registered by the container.
   class EventBusChangeRecorder {
     @Subscribe public void recordCustomerChange(ChangeEvent e) {
       recordChange(e.getChange());
     }
   }
```

> The intent is actually clearer in the second case: there's less noise code, and the event handler has a clear and meaningful name.

### What about a generic `Handler<T>` interface?
> Some have proposed a generic `Handler<T>` interface for `EventBus` listeners.  This runs into issues with Java's use of type erasure, not to mention problems in usability.

Let's say the interface looked something like the following:
```java
interface Handler<T> {
  void handleEvent(T event);
}
```

> Due to erasure, no single class can implement a generic interface more than once with different type parameters.  This is a giant step backwards from traditional Java Events, where even if `actionPerformed` and `keyPressed` aren't very meaningful names, at least you can implement both methods!

### Doesn't `EventBus` destroy static typing and eliminate automated refactoring support?
Some have freaked out about `EventBus`'s `register(Object)` and `post(Object)` methods' use of the `Object` type.

`Object` is used here for a good reason: the Event Bus library places no restrictions on the types of either your event listeners (as in `register(Object)`) or the events themselves (in `post(Object)`).

Event handler methods, on the other hand, must explicitly declare their argument type -- the type of event desired (or one of its supertypes).  Thus, searching for references to an event class will instantly find all handler methods for that event, and renaming the type will affect all handler methods within view of your IDE (and any code that creates the event).

It's true that you can rename your `@Subscribed` event handler methods at will; Event Bus will not stop this or do anything to propagate the rename because, to Event Bus, the names of your handler methods are irrelevant.  Test code that calls the methods directly, of course, will be affected by your renaming -- but that's what your refactoring tools are for.  We see this as a feature, not a bug: being able to rename your handler methods at will lets you make their meaning clearer.

### What happens if I `register` a listener without any handler methods?
Nothing at all.

The Event Bus was designed to integrate with containers and module systems, with Guice as the prototypical example.  In these cases, it's convenient to have the container/factory/environment pass _every_ created object to an `EventBus`'s `register(Object)` method.

This way, any object created by the container/factory/environment can hook into the system's event model simply by exposing handler methods.

### What Event Bus problems can be detected at compile time?
Any problem that can be unambiguously detected by Java's type system.  For example, defining a handler method for a nonexistent event type.

### What Event Bus problems can be detected immediately at registration?
Immediately upon invoking `register(Object)` , the listener being registered is checked for the _well-formedness_ of its handler methods. Specifically, any methods marked with `@Subscribe` must take only a single argument.

Any violations of this rule will cause an `IllegalArgumentException` to be thrown.

(This check could be moved to compile-time using APT, a solution we're researching.)

### What `EventBus` problems may only be detected later, at runtime?
If a component posts events with no registered listeners, it _may_ indicate an error (typically an indication that you missed a `@Subscribe` annotation, or that the listening component is not loaded).

(Note that this is _not necessarily_ indicative of a problem.  There are many cases where an application will deliberately ignore a posted event, particularly if the event is coming from code you don't control.)

To handle such events, register a handler method for the `DeadEvent` class.  Whenever `EventBus` receives an event with no registered handlers, it will turn it into a `DeadEvent` and pass it your way -- allowing you to log it or otherwise recover.

### How do I test event listeners and their handler methods?
Because handler methods on your listener classes are normal methods, you can simply call them from your test code to simulate the `EventBus`.

## Why can't I do <magic thing> with `EventBus`?
`EventBus` is designed to deal with a large class of use cases really, really well.  We prefer hitting the nail on the head for most use cases to doing decently on all use cases.

Additionally, making `EventBus` extensible -- and making it useful and productive to extend, while _still_ allowing ourselves to make additions to the core `EventBus` API that don't conflict with any of your extensions -- is an extremely difficult problem.

If you really, really need magic thing X, that `EventBus` can't currently provide, you should file an issue, and then design your own alternative.
