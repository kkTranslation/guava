# 图

Guava的`common.graph`是一个建模图结构的函数库，也即它们之间的实体关系， 例如包括网页和超链接； 科学家和他们的论文 ；飞机场之间的路线；家庭之间的关系树。目的就是提供一个通用的可扩展性的语言来处理这些数据。

## 定义

一个图包含一组结点集合（也叫做顶点集合） 和一个边集合；任意两顶点由两条边相互连接。入射到一条边的结点叫做端点。

（当我们介绍下面的`Graph` 接口时， 我们使用“graph” 小写字母g 来表达这种数据结构， 当我们涉及库中某一具体的类型， 我们才用`Graph` ）

如果一个边有定义的开始和结束， 那么是有向边， 否则是无向边。 有向边适合建立非对称关系（“祖先于”， “链接到”， “作者为”）， 然而无向边适合建立对称关系（“和谁协作发论文”， “两者之间距离”， “兄弟姊妹关系”）。

如果一个图的所有边都是有向的，此图为有向图，如果每条边都是无向的， 此图为无向图。（`common.graph` 不支持同时存在有向图和无向图）。

示例：

```java
graph.addEdge(nodeU, nodeV, edgeUV);
```

* `nodeU` 和`nodeV` 相互连接
* `edgeUV` 入射到 `nodeU` 和 `nodeV`

如果 `graph` 是有向的， 那么：

* `nodeU` 是 `nodeV` 结点的前置结点
* `nodeV` 是 `nodeU` 的后置结点
* `edgeUV` 是 `nodeU` 的出边
* `edgeUV` 是 `nodeV` 的入边
* `nodeU` 是 `edgeUV` 源点
* `nodeV` 是 `edgeUV` 终点

如果`graph` 是无向的：

* `nodeU` 既是`	nodeV` 的前置结点也是后置结点
* `nodeV` 既是 `nodeU`的前置结点也是后置结点
* `edgeUV` 既是 `nodeU` 的入边也是出边
* `edgeUV`  既是 `nodeV`的入边也是出边

以上是关于 `graph` 的所有关系。

一个自环是一条 连接结点自身的边； 等价的， 它是一种端点都为同一结点的边。 如果自环是有向的， 那么它的入射结点既是入边也是出边， 且入射结点既是源点也是终点。

## 功能

`common.graph` 关注于提供支持处理图形的接口和类。它不提供I/O 或可视化的功能支持。它有着有限的功能选择。对这个问题的更多讨论请看`FAQ`
总之， `common.graph`  支持下面这些图形类型：

- 有向图
- 无向图
- 带边权图
- 不允许自环的图
- 不允许有重边的图（带重边图有时候叫做多重图）
- 图的结点或边是否是有序插入，或无序

这些在javadoc中被确定的类型的图被特定的`common.graph` 类型 所支持。这些图被每种graph类型的内置实现所支持  。在这个库中的实现（尤其是第三方实现），并不要求支持所有的这些图形类型（javadoc定义的）， 并且可能额外支持其它类型

这个库对于底层数据结构的选择是不可知的：
图关系可能被存储为矩阵， 邻接表， 邻接map等等， 取决于实现者想优化什么样的用例。

`common.graph`  目前不包括对下面图类型的显示的支持， 虽然它们能够使用已经存在的图类型进行建模：

- 树， 森林

* graphs with elements of the same kind \(nodes or edges\) that have different
    types \(for example: bipartite/k-partite graphs, multimodal graphs\)

* 超图

  `common.graph` 不允许图同时存在单向边和双向边

  `Graphs` 提供了一些基本的工具（例如: 复制和比较图）

## 图类型

### Graph

`Graph` 是最简单最基本的图类型，它定义了最基本的操作来处理节点之间的关系， 例如：

`successors(node)` , `adjacentNodes(node)` , `inDegree(node)`  , 结点node是独一无二的对象， 你可以把它看作是映射到内部数据结构中的键。

例如：`Graph<BuildTarget>`  代表了`BuildTarget` 实体间的依赖关系

### ValueGraph

 `ValueGraph`  是一个`Graph` 的子接口， 把任意值和边相关联， 类似于`Map`  中的值， 边值并不要求是独一无二的（结点要求）

例如：`ValueGraph<Airport, TransitTime>`  是一个有向图，每个`TransitTime`  对象   代表从源`Airport` 到 目的 `Airport` 所要求的时间。因为两个不同对飞机场之间的传输时间可能相同，这就使得 `ValueGraph` 比`Network` 有更好的适应性。

### Network

[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 是一个独立的图类型（既不是`Graph` 的子类也不是它的超类），有着所有`Graph`  拥有的节点相关的方法， 还有着独一无二的边对象， 这个唯一约束允许`Network`很 自然的支持平行边和一些处理边与边，结点与边之间关系的方法， 譬如：`outEdges(node)`, `incidentNodes(edge)` ,`edgesConnecting(nodeU, nodeV)` 

[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html)提供了一个`asGraph()` 方法， 它返回一个`Network` 的`Graph` 视图， 这允许操作`Graph` 实例的方法对[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 同样有效。

例如：`Network<Webpage, Link>` , 表示通过超链接作为媒介的网页之间的关系

### 选择合适的图类型

三种图类型的本质的区别就是它们在边的表示上。

[`Graph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graph.html)  的边是通过节点之间匿名的连接， 没有自身的标志或属性。，如果每对节点间最多有一条边， 并且不需要去和边关联任何的信息，那么你应该使用`Graph`  。

[`ValueGraph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/ValueGraph.html) 有着带权值的边， 且边权不一定是唯一的。 如果每对结点最多有一条边， 并且你需要在边上关联一些信息，并且对于不同的边，信息可能相同， 那么你应该使用`ValueGraph` 。

[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html)  有独一无二的边对象，这点像结点。如果你的边对象是唯一的， 并且你想引用它们， 那么你应该使用`Network` 。（注意这个独一无二允许`Network` 支持平行边）



## 构建图实例

`common.graph` 提供的实现类不是public的， 故意地， 这减少了用户需要知道的public类型的数目， 并且使得更加容易的驾驭内置实现提供的各种各样的功能， 而没有吓到仅仅想创建一个图的用户。

为了创建一个graph类型的内置实现的实例， 使用相应的`Builder` 类： [`GraphBuilder`](#), [`ValueGraphBuilder`](#), or [`NetworkBuilder`](#).

```java
MutableGraph<Integer> graph = GraphBuilder.undirected().build();

MutableValueGraph<City, Distance> roads = ValueGraphBuilder.directed().build();

MutableNetwork<Webpage, Link> webSnapshot = NetworkBuilder.directed()
    .allowsParallelEdges(true)
    .nodeOrder(ElementOrder.natural())
    .expectedNodeCount(100000)
    .expectedEdgeCount(1000000)
    .build();
```

* 你可以用下面两种方法之一来得到一个图的`Builder`实例：
  * 调用静态方法`directed()` 或 `undirected()`. `Builder`提供的每个图实例将会是有向的或着无向的。
  * 调用静态方法`from()`  ， 会基于一个已经存在的图实例返回一个`Builder`。
* 在你创建了你的`Builder`实例之后， 你能够进一步选择性的明确其它的特性和功能。
* 你可以在一个相同的`Builder`实例上调用`build()` 多次， 来构建多个图实例。
* 你不需要在`Builder`上明确结点和边的类型， 在图类型自身明确它们就足够了。
* `Builder`方法返回一个相关图类型的`Mutable`子类型， 提供了突变方法；更多信息，下面有解释。

### Builder constraints vs. optimization hints

`Builder` 类型正常产生两种类型的选择：约束 和 优化提示。

约束明确了由一个给定的`Builder`实例来创建的图所必须满足的行为和属性， 比如：

- 图是否是无向的
- 图是否允许自环
- 图的边是否被排序

等等。。。

优化提示被实现类选择性的用来提升性能， 例如， 确定内部数据结构的类型和初始化大小， 但它不对效果做保证。

每种图类型都对有特定constraints的`Builder`提供了响应的访问器（accessors ），但是对optimization hints 没有提供访问器

## `Mutable` 和`Immutable` 图

### `Mutable*` types

每个图类型都有一个相应的`Mutable*` 子类型：: [`MutableGraph`](#), [`MutableValueGraph`](#), 和[`MutableNetwork`](#)。  这些子类型定义了突变方法：

* 添加和移除结点的方法：
  * `addNode(node)` 和removeNode(node)`
* 添加和移除边的方法：:
  * [`MutableGraph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/MutableGraph.html)
    * `putEdge(nodeU, nodeV)`
    * `removeEdge(nodeU, nodeV)`
  * [`MutableValueGraph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/MutableValueGraph.html)
    * `putEdgeValue(nodeU, nodeV, value)`
    * `removeEdge(nodeU, nodeV)`
  * [`MutableNetwork`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/MutableNetwork.html)
    * `addEdge(nodeU, nodeV, edge)`
    * `removeEdge(edge)`

这是对java集合框架和Guava的新集合类型历史工作的违背， 每个类型都包含突变方法的签名. 我们选择分离出突变方法到子类型是去鼓励防御性编程。 通常来说， 如果你的代码仅仅是检查或遍历一个图，且不去改变它， 它的输入应该在[`Graph`](#), [`ValueGraph`](#), or [`Network`](#) 上被指定， 而不是他们的突变子类型。 另一方面， 如果你的代码需要改变一个对象。那么你的代码就不得不和标有“Mutable”的类型。

因为Graph 等是接口， 尽管它们不包含突变方法，提供一个这个接口的实例给调用者并不能保证将不会被调用者所改变， 因为（如果事实上是一个Mutable*子类型）调用者能够强转到这个子类型。如果你想提供一个保证， 一个图的方法参数和返回类型不能够被修改， 你应该使用Immutable实现。

### `Immutable*` implementations

每个图都有相应的 `Immutable` implementation. 实现 。这些类和Guava's `ImmutableSet`, `ImmutableList`,  
`ImmutableMap`,等等相似: 一旦被构建， 便不能被修改， 且它们内部使用不可突变的数据结构。

不像其它 Guava `Immutable` 类型,然而， 这些实现没有任何突变方法， 所以它们对试图修改不需要抛出
`UnsupportedOperationException` 。

你能够通过他的静态方法`copyOf()` 来创建一个 `ImmutableGraph`实例：

```java
ImmutableGraph<Integer> immutableGraph = ImmutableGraph.copyOf(graph);
```

#### 保证

每个`Immutable*`类型做出如下保证：

- 不变性：元素不能被添加、删除或替换（这些类没有实现`Mutable*`接口）
- 确定性的迭代： 迭代顺序和图输入顺序相同
- [**线程安全**](#synchronization): ：多个线程并发访问图是线程安全的
- 完整性：此类型在外部package中不能产生子类（使得这些保证被孤立）



### 把这些类当作“接口”而不是实现

每个`Immutable*`类都是一个提供了有意义的行为保证的类型， 不仅仅是一个特定的实现， 你更应该像接口一样对待它们。

字段和方法返回值存储在一个`Immutable*`实例（比如`ImmutableGraph`）中，　应该被声明为 `Immutable*` 类型之一，　而不是相应的接口类型（比如 `Graph`）。　这向调用者表达了上面列举的所有语义保证。

另一方面，　一个`ImmutableGraph`类型的参数通常对调用者来说是一种厌恶。相反，　接受`Graph`.

警告：　就像 [到处](#graph-elements-nodes-and-edges)标注的一样，　当修改一个存在集合中的元素（用一种影响它的 `equals()`行为的方式）总是一个坏主意。将会导致未知的行为和bug。通常最好避免使用突变对象作为一个`Immutable*`实例的元素，且很多使用者期望`Immutable*`对象是绝不可变的。



## 图元素（结点和边）

****

**`Network`**中的结点就像**`Map`**中的key一样被使用。 即：

* 它们在图中必须是独一无二的`nodeA` and `nodeB` 被认为是不同的
  如果  `nodeA.equals(nodeB) == false`.
* 它们必须有合适的且一致的 `equals()`
    和`hashCode()`方法实现
* 如果元素被排序 \(例如通过`GraphBuilder.orderNodes()`\), 顺序必须和`Comparator` and
    `Comparable` 接口的规定具有一致性

如果图的元素有突变状态：

* 突变状态不能被反射在`equals()/hashCode()` 方法上
  \(这在`Map` 文档中被详细讨论\)

* 不要构建多个相同的元素并期望它们是可互换的. 特别的， 当向一个图中添加一些元素， 你应该创建元素一次， 并应该存储它们的引用， 免得在创建过程中多次引用到此元素（而不是每次都传递`new MyMutableNode(id)`到每个`add*()` 调用中）

  ​

如果你需要保存每个可突变元素的状态， 一个选择就是使用不可突变元素在一个独立的数据结构中保存突变状态。

图类型元素必须非null

## 函数库约定以及行为

这部分章节讨论`common.graph`类型的一些内置实现的行为。

### 突变

你能够添加一条边，即使它的结点之前还没有被添加到图中， 如果之前结点不存在， 结点会“默默地”被加入图中

```java
Graph<Integer> graph = GraphBuilder.directed().build();  // graph is empty
graph.putEdge(1, 2);  // this adds 1 and 2 as nodes of this graph, and puts
                      // an edge between them
if (graph.nodes().contains(1)) {  // evaluates to "true"
  ...
}
```

### `equals()`, `hashCode()`, 和图等价

`common.graph`的图实现定义`equals()` 依据引用相等： 也就是说， 如果两个图是同一个对象， 两个图会被看作等价。虽然这是`Object.equals()`的默认行为， 在集合类型上通常不这样。 然而既然我们有不同的潜在的规则来比较图（例如： 是否考虑边对象/值， 或着仅仅考虑图的拓扑？），我们决定从图内容的比较上分离`equals()`的行为。

相似的， `hashCode()` 被定义在我们的实现中，作为`System.identityHashCode(this)` 的输出。默认的`Object.hashCode()`的行为。

如果你想基于它们的结构属性来比较两个图， 使用[`Graphs`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Graphs.html)`.equivalent()`  工具； 在`ValueGraph`, 和`Network`  中均有重载。

`ValueGraph`, 和`Network` 作为参数.

```java
Graph<Integer> graph1, graph2;
ValueGraph<Integer, Double> valueGraph1, valueGraph2;
Network<Integer, MyEdge> network1, network2;
...
// compare based on node relationships only
if (Graphs.equivalent(graph1, graph2)) { ... }
if (Graphs.equivalent((Graph) valueGraph1, (Graph) valueGraph2)) { ... }
if (Graphs.equivalent(network1.asGraph(), network2.asGraph())) { ... }

// compare based on node relationships and edge values
if (Graphs.equivalent(valueGraph1, valueGraph2)) { ... }

// compare based on node relationships and edge identities
if (Graphs.equivalent(network1, network2)) { ... }
```

### 访问器方法

返回集合的访问器:

* 可能返回graph的视图， 对会影响视图的图修改不被支持，并可能会抛出`ConcurrentModificationException`。（例如：在通过`nodes()`进行遍历图时调用`addNode(n)` 或`removeNode(n)` 方法）
* 如果输入有效，但是没有元素满足这个请求（例如：`adjacentNodes(node)` ）将会返回一个空的集合, 如果`node` 结点没有邻接点。

访问器将会抛出 `IllegalArgumentException` 异常， 如果传递一个并不在图中的结点元素。

访问器采用 `Object`参数而不是Java集合框架建立的泛型来匹配模式（ match  
the pattern ）。

### 同步

这取决于每个图的实现来决定它本身的同步策略。默认地， 未定义的行为可能导致被另一个线程对任一方法的调用。

通常来说，内置的突变实现提供非同步的保证， 但是`Immutable*` 类是线程安全的。

### 元素对象

你所添加到图中的结点， 边， 值对象是和内置的类实现是无关的； 它们仅仅用于内部数据结构的key。 这意味着结点、边可能在多个图实例之间共享。

默认的， 结点和边对象是插入有序的（也就是说，用 `Iterator`来对`nodes()` 和`edges()` 来访问，是按照它们所添加到图中的顺序， 就像`LinkedHashSet`）。

## Notes for implementors

### Storage models

`common.graph` 提供了多种机制来存储图的拓扑，包括：

* 图的实现存储拓扑（例如，通过存储`Map<N, Set<N>>`， 映射结点到它们的邻接点）；这暗示结点仅仅是key， 可以在多图间共享。
* 结点存储拓扑（例如， 通过存储 邻接点到`List<E>` ）； 这意味着结点是graph-specific
* 一个分离的数据仓库（例如， 一个数据库）来存储拓扑

注意： `Multimap`是一个不充分的数据结构，对于图的可以支持孤立结点的实现。由于它的限制（`Multimap`）， 一个key要不映射到至少一个值， 要不不出现在`Multimap`中。

### 访问器行为

对于返回一个集合的访问器， 对于此语义有一些选择：

1、集合是不可变的拷贝（`ImmutableSet`等）：尝试用任何方法来修改集合都会抛出异常， 对图的修改将不会反应在集合中

2、集合是不可修改的视图（`Collections.unmodifiableSet()`等）：尝试用任何方法来修改集合都会抛出异常， 对图的修改将会反应在集合中

3、集合是可变的拷贝， 它可能被修改， 但是对于集合的修改并不会反应在图中， 反之亦然。

4、集合是可变的视图， 它可能被修改， 但是对于集合的修改并不会反应在图中， 反之亦然。



\(In theory one could return a collection which passes through writes in one  
direction but not the other \(collection-to-graph or vice-versa\), but this is  
basically never going to be useful or clear, so please don't. :\) \)

\(1\) 和(2\) 通常是较倾向的选择; 内置实现通常使用 \(2\).

(3) 是一个可工作的选择， 但是会对使用者造成迷惑， 如果他们期望修改将会影响视图， 或着对图的修改将会反应在集合中。

\(4\)是一个冒险的设计选择， 并且应该非常谨慎的使用，因为保持内部数据结构的一致性不容易。

### `Abstract*` 类

每个图类型都有相应的 `Abstract` 类: `AbstractGraph`, 等.

图接口的实现者应该， 如果可能的话， 继承合适的抽象类而不是直接实现接口。抽象类提供了一些关键方法的实现，或着可以让子类有一致的某些方法的实现，例如：

* `*degree()`
* `toString()`
* `Graph.edges()`
* `Network.asGraph()`

## 代码示例

### 此 `node` 在图中？

```java
graph.nodes().contains(node);
```

### 在结点 `u` 和`v` 之间是否有边\(根据现图中已知)?

```java
graph.successors(u).contains(v);
```

### 基本的`Graph` 示例

```java
MutableGraph<Integer> graph = GraphBuilder.directed().build();
graph.addNode(1);
graph.putEdge(2, 3);  // also adds nodes 2 and 3 if not already present

Set<Integer> successorsOfTwo = graph.successors(2); // returns {3}

graph.putEdge(2, 3);  // no effect; Graph does not support parallel edges
```

### 基本的 [`ValueGraph`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/ValueGraph.html) 示例

```java
MutableValueGraph<Integer, Double> weightedGraph = ValueGraphBuilder.directed().build();
weightedGraph.addNode(1);
weightedGraph.putEdgeValue(2, 3, 1.5);  // also adds nodes 2 and 3 if not already present
weightedGraph.putEdgeValue(3, 5, 1.5);  // edge values (like Map values) need not be unique
...
weightedGraph.putEdgeValue(2, 3, 2.0);  // updates the value for (2,3) to 2.0
```

### 基本的[`Network`](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/graph/Network.html) 示例

```java
MutableNetwork<Integer, String> network = NetworkBuilder.directed().build();
network.addNode(1);
network.addEdge("2->3", 2, 3);  // also adds nodes 2 and 3 if not already present

Set<Integer> successorsOfTwo = network.successors(2);  // returns {3}
Set<String> outEdgesOfTwo = network.outEdges(2);   // returns {"2->3"}

network.addEdge("2->3 too", 2, 3);  // throws; Network disallows parallel edges
                                    // by default
network.addEdge("2->3", 2, 3);  // no effect; this edge is already present
                                // and connecting these nodes in this order

Set<String> inEdgesOfFour = network.inEdges(4); // throws; node not in graph
```

### 遍历无向图

```java
// Return all nodes reachable by traversing 2 edges starting from "node"
// (ignoring edge direction and edge weights, if any, and not including "node").
Set<N> getTwoHopNeighbors(Graph<N> graph, N node) {
  Set<N> twoHopNeighbors = new HashSet<>();
  for (N neighbor : graph.adjacentNodes(node)) {
    twoHopNeighbors.addAll(graph.adjacentNodes(neighbor));
  }
  twoHopNeighbors.remove(node);
  return twoHopNeighbors;
}
```

### 遍历有向图

```java
// Update the shortest-path weighted distances of the successors to "node"
// in a directed Network (inner loop of Dijkstra's algorithm)
// given a known distance for {@code node} stored in a {@code Map<N, Double>},
// and a {@code Function<E, Double>} for retrieving a weight for an edge.
void updateDistancesFrom(Network<N, E> network, N node) {
  double nodeDistance = distances.get(node);
  for (E outEdge : network.outEdges(node)) {
    N target = network.target(outEdge);
    double targetDistance = nodeDistance + edgeWeights.apply(outEdge);
    if (targetDistance < distances.getOrDefault(target, Double.MAX_VALUE)) {
      distances.put(target, targetDistance);
    }
  }
}
```

## FAQ

### 为什么Guava引入 `common.graph`?

The same arguments apply to graphs as to many other things that Guava does:

* code reuse/interoperability/unification of paradigms: lots of things relate
  to graph processing
* efficiency: how much code is using inefficient graph representations? too
    much space \(e.g. matrix representations\)?
* correctness: how much code is doing graph analysis wrong?
* promotion of use of graph as ADT: how many people would be working with
    graphs if it were easy?
* simplicity: code which deals with graphs is easier to understand if it’s
    explicitly using that metaphor.

### What kinds of graphs does `common.graph` support?

This is answered in the ["Capabilities"](#capabilities) section, above.

### `common.graph` doesn’t have feature/algorithm X, can you add it?

Maybe. You can email us at `guava-discuss@googlegroups.com` or [open an issue on  
GitHub](https://github.com/google/guava/issues). 

Our philosophy is that something should only be part of Guava if \(a\) it fits in  
with Guava’s core mission and \(b\) there is good reason to expect that it will be  
reasonably widely used.

`common.graph` will probably never have capabilities like visualization and I/O;  
those would be projects unto themselves and don’t fit well with Guava’s mission.

Capabilities like traversal, filtering, or transformation are better fits, and  
thus more likely to be included, although ultimately we expect that other graph  
libraries will provide most capabilities.

### Does it support very large graphs \(i.e., MapReduce scale\)?

Not at this time. Graphs in the low millions of nodes should be workable, but  
you should think of this library as analogous to the Java Collections Framework  
types \(`Map`, `List`, `Set`, and so on\).

### Why should I use it instead of something else?

**tl;dr**: you should use whatever works for you, but please let us know what  
you need if this library doesn't support it!

The main competitors to this library \(for Java\) are: [JUNG](http://jung.sf.net)  
and [JGraphT](http://jgrapht.org/).

`JUNG` was co-created by Joshua O'Madadhain \(the `common.graph` lead\) in 2003,  
and he still maintains it. JUNG is fairly mature and full-featured and is widely  
used, but has a lot of cruft and inefficiencies. Now that `common.graph` has  
been released externally, he plans to create a new version of `JUNG` which uses  
`common.graph` for its data model.

`JGraphT` is another third-party Java graph library that’s been around for a  
while. We're not as familiar with it, so we can’t comment on it in detail, but  
it has at least some things in common with `JUNG`.

Rolling your own solution is sometimes the right answer if you have very  
specific requirements. But just as you wouldn’t normally implement your own hash  
table in Java \(instead of using `HashMap` or `ImmutableMap`\), you should  
consider using `common.graph` \(or, if necessary, another existing graph library\)  
for all the reasons listed above.

## Major Contributors

`common.graph` has been a team effort, and we've had help from a number of  
people both inside and outside Google, but these are the people that have had  
the greatest impact.

* **Omar Darwish** did a lot of the early implementations, and set the
  standard for the test coverage.
* [**James Sexton**](https://github.com/bezier89) has been the single most
    prolific contributor to the project and has had a significant influence on
    its direction and its designs. He's responsible for some of the key
    features, and for the efficiency of the implementations that we provide.
* [**Joshua O'Madadhain**](https://github.com/jrtom) started the
    `common.graph` project after reflecting on the strengths and weaknesses of
    [JUNG](http://jung.sf.net), which he also helped to create. He leads the
    project, and has reviewed or written virtually every aspect of the design
    and the code.



