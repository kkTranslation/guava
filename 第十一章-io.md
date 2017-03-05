# `ByteStreams` and `CharStreams`
Guava 使用术语”流” 来表示可关闭的，并且在底层资源中有位置状态的 I/O 数据流。术语”字节流”指的是 `InputStream` 或 `OutputStream`，”字符流”指的是 `Reader` 或 `Writer`（虽然他们的接口 `Readable` 和 `Appendable` 被更多地用于方法参数）。相应的工具方法分别在 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html'><code>ByteStreams</code></a> 和 <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html'><code>CharStreams</code></a>中。

大多数 Guava 流工具一次处理一个完整的流，并且/或者为了效率自己处理缓冲。还要注意到，接受流为参数的Guava 方法不会关闭这个流：关闭流的职责通常属于打开流的代码块。

这些类提供的一些方法包括：

| **`ByteStreams`**                        | **`CharStreams`**                        |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#toByteArray(java.io.InputStream)'><code>byte[] toByteArray(InputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#toString(java.lang.Readable)'><code>String toString(Readable)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#readLines(java.lang.Readable)'><code>List&lt;String&gt; readLines(Readable)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#copy(java.io.InputStream, java.io.OutputStream)'><code>long copy(InputStream, OutputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#copy(java.lang.Readable, java.lang.Appendable)'><code>long copy(Readable, Appendable)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#readFully(java.io.InputStream, byte[])'><code>void readFully(InputStream, byte[])</code></a> | N/A                                      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#skipFully(java.io.InputStream, long)'><code>void skipFully(InputStream, long)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#skipFully(java.io.Reader, long)'><code>void skipFully(Reader, long)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteStreams.html#nullOutputStream()'><code>OutputStream nullOutputStream()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharStreams.html#nullWriter()'><code>Writer nullWriter()</code></a> |

#### 关于 `InputSupplier` 和 `OutputSupplier` 要注意

在 `ByteStreams`、`CharStreams` 以及 `com.google.common.io` 包中的一些其他类中，某些方法仍然在使用 `InputSupplier` 和 `OutputSupplier` 接口。这两个借口和相关的方法是不推荐使用的：它们已经被下面描述的 source 和 sink 类型取代了，并且最终会被移除。

# 源与汇

通常创建I / O实用程序方法，帮助您在执行基本操作时避免处理流。例如，Guava 有 `Files.toByteArray(File)` 和 `Files.write(File, byte[])`。然而，流工具方法的创建经常最终导致散落各处的相似方法，每个方法读取不同类型的数据源或sink。例如，Guava 中的`Resources.toByteArray(URL)`和 `Files.toByteArray(File)`做了同样的事情，只不过数据源一个是 URL，一个是文件。

为了解决这个问题，Guava 有一系列关于源与汇的抽象。源或汇指某个你知道如何从中打开流的资源，比如 `File`或 `URL`。源是可读的，汇是可写的。此外，源与汇按照字节和字符划分类型。

|             | **字节**                                | **字符**                                |
| :---------- | :--------------------------------------- | :--------------------------------------- |
| **读** | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html'><code>ByteSource</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html'><code>CharSource</code></a> |
| **写** | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html'><code>ByteSink</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html'><code>CharSink</code></a> |

这些API的优点是它们提供了一组通用的操作。 例如，将数据源打包为`ByteSource`后，无论该源是什么，您都将获得相同的一组方法。

### 创建源与汇

Guava 提供了若干源与汇的实现：

| **字节**                                | **字符**                                |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSource(java.io.File)'><code>Files.asByteSource(File)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSource(java.io.File, java.nio.charset.Charset)'><code>Files.asCharSource(File, Charset)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asByteSink(java.io.File, com.google.common.io.FileWriteMode...)'><code>Files.asByteSink(File, FileWriteMode...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#asCharSink(java.io.File, java.nio.charset.Charset, com.google.common.io.FileWriteMode...)'><code>Files.asCharSink(File, Charset, FileWriteMode...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asByteSource(java.net.URL)'><code>Resources.asByteSource(URL)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Resources.html#asCharSource(java.net.URL, java.nio.charset.Charset)'><code>Resources.asCharSource(URL, Charset)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#wrap(byte[])'><code>ByteSource.wrap(byte[])</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#wrap(java.lang.CharSequence)'><code>CharSource.wrap(CharSequence)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#concat(com.google.common.io.ByteSource...)'><code>ByteSource.concat(ByteSource...)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#concat(com.google.common.io.CharSource...)'><code>CharSource.concat(CharSource...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#slice(long, long)'><code>ByteSource.slice(long, long)</code></a> | N/A                                      |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#asCharSource(java.nio.charset.Charset)'><code>ByteSource.asCharSource(Charset)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#asCharSink(java.nio.charset.Charset)'><code>ByteSink.asCharSink(Charset)</code></a> |

此外，你也可以继承这些类，以创建新的实现。

注：把已经打开的流（比如 `InputStream`）包装为源或汇听起来是很有诱惑力的，但是应该避免这样做。源与汇的实现应该在每次 `openStream()`方法被调用时都创建一个新的流。始终创建新的流可以让源或汇管理流的整个生命周期，并且让多次调用 `openStream()`返回的流都是可用的。此外，如果你在创建源或汇之前创建了流，你不得不在异常的时候自己保证关闭流，这压根就违背了发挥源与汇 API 优点的初衷。

### 使用源与汇

一旦有了源与汇的实例，就可以进行若干读写操作。

#### 通用操作

所有源与汇都有一些方法用于打开新的流用于读或写。默认情况下，其他源与汇操作都是先用这些方法打开流，然后做一些读或写，最后保证流被正确地关闭了。这些方法列举如下：

* `openStream()` - 根据源与汇的类型，返回`InputStream`、`OutputStream`、`Reader` 或者`Writer`。
* `openBufferedStream()` - 根据源与汇的类型，返回 `InputStream`、`OutputStream`、`BufferedReader `或
者 `Writer`。返回的流保证在必要情况下做了缓冲。例如，从字节数组读数据的源就没有必要再在内存中作缓冲，这就是为什么该方法针对字节源不返回 `BufferedInputStream`。字符源属于例外情况，它一定返回 `BufferedReader`，因为 `BufferedReader` 中才有 `readLine()`方法。

#### 源操作

| **`字节源`**                         | **`字符源`**                         |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#read()'><code>byte[] read()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#read()'><code>String read()</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readLines()'><code>ImmutableList&lt;String&gt; readLines()</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#readFirstLine()'><code>String readFirstLine()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo(com.google.common.io.ByteSink)'><code>long copyTo(ByteSink)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo(com.google.common.io.CharSink)'><code>long copyTo(CharSink)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#copyTo(java.io.OutputStream)'><code>long copyTo(OutputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#copyTo(java.lang.Appendable)'><code>long copyTo(Appendable)</code> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#size()'><code>long size()</code></a> (in bytes) | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#length--'><code>long length()</code></a> (in chars) |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#isEmpty()'><code>boolean isEmpty()</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSource.html#isEmpty()'><code>boolean isEmpty()</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#contentEquals(com.google.common.io.ByteSource)'><code>boolean contentEquals(ByteSource)</code></a> | N/A                                      |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSource.html#hash(com.google.common.hash.HashFunction)'><code>HashCode hash(HashFunction)</code></a> | N/A                                      |

#### 汇操作

| **`字节汇`**                           | **`字符汇`**                           |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#write(byte[])'><code>void write(byte[])</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#write(java.lang.CharSequence)'><code>void write(CharSequence)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/ByteSink.html#writeFrom(java.io.InputStream)'><code>long writeFrom(InputStream)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeFrom(java.lang.Readable)'><code>long writeFrom(Readable)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable)'><code>void writeLines(Iterable&lt;? extends CharSequence&gt;)</code></a> |
| N/A                                      | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/CharSink.html#writeLines(java.lang.Iterable, java.lang.String)'><code>void writeLines(Iterable&lt;? extends CharSequence&gt;, String)</code></a> |

### 范例

```java
// Read the lines of a UTF-8 text file
ImmutableList<String> lines = Files.asCharSource(file, Charsets.UTF_8)
    .readLines();

// Count distinct word occurrences in a file
Multiset<String> wordOccurrences = HashMultiset.create(
  Splitter.on(CharMatcher.WHITESPACE)
    .trimResults()
    .omitEmptyStrings()
    .split(Files.asCharSource(file, Charsets.UTF_8).read()));

// SHA-1 a file
HashCode hash = Files.asByteSource(file).hash(Hashing.sha1());

// Copy the data from a URL to a file
Resources.asByteSource(url).copyTo(Files.asByteSink(file));
```

# `Files`

除了创建文件源和文件的方法，`Files` 类还包含了若干你可能感兴趣的便利方法。

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#createParentDirs(java.io.File)'><code>createParentDirs(File)</code></a> | 必要时为文件创建父目录 |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getFileExtension(java.lang.String)'><code>getFileExtension(String)</code></a> | 返回给定路径所表示文件的扩展名 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#getNameWithoutExtension(java.lang.String)'><code>getNameWithoutExtension(String)</code></a> | 返回去除了扩展名的文件名 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#simplifyPath(java.lang.String)'><code>simplifyPath(String)</code></a> | 规范文件路径，并不总是与文件系统一致，请仔细测试 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/io/Files.html#fileTreeTraverser()'><code>fileTreeTraverser()</code></a> | 返回 TreeTraverser 用于遍历文件树 |
