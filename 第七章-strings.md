# Joiner
用分隔符把字符串序列连接起来也可能会遇上不必要的麻烦。如果字符串序列中含有 null，那连接操作会更难。Fluent 风格的  <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Joiner.html'><code>Joiner</code></a> 让连接字符串更简单。

```java

Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```
上述代码返回 "Harry; Ron; Hermione".  另外， 而不是像 `skipNulls`那样直接忽略null。  `useForNull(String)`方法可以给某个字符串来替换null。

 `Joiner`也可以用来连接对象类型，在这种情况下，它会把对象的  `toString()` 值连接起来。

```java

Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

**警告:**joiner 实例总是不可变的。用来定义 `Joiner`目标语义的配置方法总会返回一个新的 `Joiner`实例。这使得 `Joiner` 实例都是线程安全的，你可以将其定义为`static final`常量。

# 拆分器[Splitter]
JDK 内建的字符串拆分工具有一些古怪的特性。比如，String.split 悄悄丢弃了尾部的分隔符。 问

题: `",a,,b,".split(",")` 返回？
  1. `"", "a", "", "b", ""`
  2. `null, "a", null, "b", null`
  3. `"a", null, "b"`
  4. `"a", "b"`
  5. None of the above

正确答案是 5: `"", "a", "", "b"`. 只有尾部的空字符串被忽略了.  What is this I don't even.

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html'><code>Splitter</code></a> 使用令人放心的、直白的流畅 API 模式对这些混乱的特性作了完全的掌控。

```java

Splitter.on(',')
       .trimResults()
       .omitEmptyStrings()
       .split("foo,bar,,   qux");
```
上述代码返回`Iterable<String>` 其中包含 "foo", "bar", "qux".   `Splitter` 可以被设置为按照任何模式、字符、字符串或字符匹配器拆分。

### 拆分器工厂
| Method                                   | Description | Example            |
| :--------------------------------------- | :---------- | :----------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(char)'><code>Splitter.on(char)</code></a> | 按单个字符拆分     | `Splitter.on(';')` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(com.google.common.base.CharMatcher)'><code>Splitter.on(CharMatcher)</code></a> |  按字符匹配器拆分 | `Splitter.on(CharMatcher.BREAKING_WHITESPACE)`<br><code>Splitter.on(CharMatcher.anyOf(";,."))</code>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(java.lang.String)'><code>Splitter.on(String)</code></a> </td><td> 按照字符串拆分. </td><td> <code>Splitter.on(", ")</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on(java.util.regex.Pattern)'><code>Splitter.on(Pattern)</code></a><br><a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#onPattern(java.lang.String)'><code>Splitter.onPattern(String)</code></a> </td><td>按照正则表达式拆分. </td><td> <code>Splitter.onPattern("\r?\n")</code> </td></tr>
<tr><td> <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#fixedLength(int)'><code>Splitter.fixedLength(int)</code></a> </td><td>  按固定长度拆分；最后一段
可能比给定长度短，但不会为空. </td><td> <code>Splitter.fixedLength(3)</code> </td></tr></tbody></table>

### 拆分器修饰符
| Method                                   | Description                              | Example                                  |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#omitEmptyStrings()'><code>omitEmptyStrings()</code></a> | 从结果中自动忽略空字符串                             | `Splitter.on(',').omitEmptyStrings().split("a,,c,d")` returns `"a", "c", "d"` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults()'><code>trimResults()</code></a> | 移除结果字符串的前导空白和尾部空白 等价于trimResults(CharMatcher.WHITESPACE)`. | `Splitter.on(',').trimResults().split("a, b, c, d")` returns `"a", "b", "c", "d"` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults(com.google.common.base.CharMatcher)'><code>trimResults(CharMatcher)</code></a> | 给定匹配器，移除结果字符串的前导匹配字符和尾部匹配字符              | `Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")` returns `"a ", "b_ ", "c"`. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#limit(int)'><code>limit(int)</code></a> | 限制拆分出的字符串数量，到达数量自动停止拆分                   | `Splitter.on(',').limit(3).split("a,b,c,d")` returns `"a", "b", "c,d"` |

TODO: Map splitters

如果你想要拆分器返回`List`,只要使用 `Lists.newArrayList(splitter.split(string))` 或类似方法。

**警告: **`Splitter`实例总是不可变的。用来定义 `Splitter`目标语义的配置方法总会返回一个新的 `Splitter`实例。这使得 `Splitter`实例都是线程安全的，你可以将其定义为 `static final` 常量。

<!--
<a href='Hidden comment: 
= Escaper =
Escaping strings correctly -- converting them into a format safe for inclusion in e.g. an XML document or a Java source file -- can be a tricky business, and critical for security reasons.  Guava provides a flexible API for escaping text, and a number of built-in escapers, in the com.google.common.escape package.

All escapers in Guava extend the [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/Escaper.html Escaper] abstract class, and support the method String escape(String).  Built-in Escaper instances can be found in several classes, depending on your needs: [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/html/HtmlEscapers.html HtmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/xml/XmlEscapers.html XmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/SourceCodeEscapers.html SourceCodeEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/net/UriEscapers.html UriEscapers], or you can build your own with [http://google.github.io/guava/releases/snapshot/api/docs/ an Escapers.Builder].  To inspect an Escaper, you can use Escapers.computeReplacement to find the replacement string for a given character.
'></a>
-->

# 字符匹配器[CharMatcher]
在以前的 Guava 版本中，`StringUtil`类疯狂地膨胀，其拥有很多处理字符串的方法:

|                  |                  |                        |                      |                |
| :--------------- | :--------------- | :--------------------- | :------------------- | :------------- |
| `allAscii`       | `collapse`       | `collapseControlChars` | `collapseWhitespace` | `indexOfChars` |
| `lastIndexNotOf` | `numSharedChars` | `removeChars`          | `removeCrLf`         | `replaceChars` |
| `retainAllChars` | `strip`          | `stripAndCollapse`     | `stripNonDigits`     |                |

所有这些方法指向两个概念上的问题:

  1. 怎么才算匹配字符？
  2. 如何处理这些匹配字符？

为了收拾这个泥潭，我们开发了  `CharMatcher`。

直观上，你可以认为一个  `CharMatcher`实例代表着某一类字符，如数字或空白字符。事实上来说， `CharMatcher` 实例就是对字符的布尔判断——CharMatcher 确实也实现了 [[Predicate<Character>|FunctionalExplained#predicate]]  -- 但类似”所有空白字符”或”所
有小写字母”的需求太普遍了，Guava 因此创建了这一 API。
然而使用 `CharMatcher` 的好处更在于它提供了一系列方法，让你对字符作特定类型的操作：修剪[trim]、折叠[collapse]、移除[remove]、保留[retain]等等。`CharMatcher` 实例首先表达了概念 1：怎么才算匹配字符？然后它还提供了很多操作回答了概念 2：如何处理这些匹配字符？这样的设计使得 API 复杂度的线性增加可以带来灵活性和功
能两方面的增长。

```
String noControl = CharMatcher.JAVA_ISO_CONTROL.removeFrom(string); // remove control characters
String theDigits = CharMatcher.DIGIT.retainFrom(string); // only the digits
String spaced = CharMatcher.WHITESPACE.trimAndCollapseFrom(string, ' ');
  // trim whitespace at ends, and replace/collapse whitespace into single spaces
String noDigits = CharMatcher.JAVA_DIGIT.replaceFrom(string, "*"); // star out all digits
String lowerAndDigit = CharMatcher.JAVA_DIGIT.or(CharMatcher.JAVA_LOWER_CASE).retainFrom(string);
  // eliminate all characters that aren't digits or lowercase
```

**Note:** `CharMatcher` deals only with `char` values; it does not understand supplementary Unicode code points in the range 0x10000 to 0x10FFFF. Such logical characters are encoded into a `String` using surrogate pairs, and a `CharMatcher` treats these just as two separate characters.

**注:** `CharMatcher` 只处理 `char` 类型代表的字符；它不能理解 0x10000 到 0x10FFFF 的 Unicode 增补字符。这些逻辑字符以代理对[surrogate pairs]的形式编码进字符串，而 `CharMatcher` 只能将这种逻辑字符看待成两个独立的字符。

## 获取字符匹配器
通过提供的`CharMatcher` 常量 很多需求能够被满足：

|                                          |                                          |                                          |                                          |                                          |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ANY'><code>ANY</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#NONE'><code>NONE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#WHITESPACE'><code>WHITESPACE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#BREAKING_WHITESPACE'><code>BREAKING_WHITESPACE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#INVISIBLE'><code>INVISIBLE</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#DIGIT'><code>DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LETTER'><code>JAVA_LETTER</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_DIGIT'><code>JAVA_DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LETTER_OR_DIGIT'><code>JAVA_LETTER_OR_DIGIT</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_ISO_CONTROL'><code>JAVA_ISO_CONTROL</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_LOWER_CASE'><code>JAVA_LOWER_CASE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#JAVA_UPPER_CASE'><code>JAVA_UPPER_CASE</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ASCII'><code>ASCII</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#SINGLE_WIDTH'><code>SINGLE_WIDTH</code></a> |                                          |

其他获取字符匹配器`CharMatcher` 的常见方法包括：

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#anyOf(java.lang.CharSequence)'><code>anyOf(CharSequence)</code></a> | 明确了所有你希望匹配的字符，例如, `CharMatcher.anyOf("aeiou")匹配给定的小写元音字符 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#is(char)'><code>is(char)</code></a> | 给定单一字符匹配                                 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#inRange(char, char)'><code>inRange(char, char)</code></a> | 给定字符范围匹配，如`CharMatcher.inRange('a', 'z')`. |

此外, `CharMatcher` 还有 <code>negate()</code></a>, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#and(com.google.common.base.CharMatcher)'><code>and(CharMatcher)</code></a>, and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#or(com.google.common.base.CharMatcher)'><code>or(CharMatcher)</code></a>.  这些在 `CharMatcher`上的简单布尔操作.

## 使用字符匹配器

`CharMatcher` 提供了多种多样的方法操作 `CharSequence`中的特定字符.  提供的方法远不止这些, 其中最常用的罗列如下：:

| Method                                   | Description                              |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#collapseFrom(java.lang.CharSequence, char)'><code>collapseFrom(CharSequence, char)</code></a> | 把每组连续的匹配字符替换为特定字符。如`WHITESPACE.collapseFrom(string, ' ')`  把字符串中的连续空白字符替换为单个空格。 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#matchesAllOf(java.lang.CharSequence)'><code>matchesAllOf(CharSequence)</code></a> | 测试是否字符序列中的所有字符都匹配。例如， `ASCII.matchesAllOf(string)` 测试是否所有字符串中的字符都在ASCII中 |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#removeFrom(java.lang.CharSequence)'><code>removeFrom(CharSequence)</code></a> | 从字符序列中移除所有匹配字符。                          |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#retainFrom(java.lang.CharSequence)'><code>retainFrom(CharSequence)</code></a> | 在字符序列中保留匹配字符，移除其他字符。                     |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#trimFrom(java.lang.CharSequence)'><code>trimFrom(CharSequence)</code></a> | 移除字符序列的前导匹配字符和尾部匹配字符。                    |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#replaceFrom(java.lang.CharSequence, java.lang.CharSequence)'><code>replaceFrom(CharSequence, CharSequence)</code></a> | 用特定字符序列替代匹配字符。                           |

所有这些方法返回  `String`，除了 `matchesAllOf`返回的是 `boolean`。 

# 字符集[Charsets]
不要这样进行字符集处理：

```java

try {
  bytes = string.getBytes("UTF-8");
} catch (UnsupportedEncodingException e) {
  // how can this possibly happen?
  throw new AssertionError(e);
}
```

试试这样

```java

bytes = string.getBytes(Charsets.UTF_8);
```

<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Charsets.html'><code>Charsets</code></a> 针对所有 Java 平台都要保证支持的六种字符集提供了常量引用。尝试使用这些常量，而不是通过名称获取字符集实例 

TODO: an explanation of charsets and when to use them

(注: 如果你在使用JDK7, 你要使用的常量在 <a href='http://docs.oracle.com/javase/7/docs/api/java/nio/charset/StandardCharsets.html'><code>StandardCharsets</code></a> !)

# 大小写格式[CaseFormat]
`CaseFormat` 被用来方便地在各种 ASCII 大小写规范间转换字符串——比如，编程语言的命名规范。`CaseFormat` 支持的格式如下： 

| Format                                   | Example            |
| :--------------------------------------- | :----------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_CAMEL'><code>LOWER_CAMEL</code></a> | `lowerCamel`       |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_HYPHEN'><code>LOWER_HYPHEN</code></a> | `lower-hyphen`     |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_UNDERSCORE'><code>LOWER_UNDERSCORE</code></a> | `lower_underscore` |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_CAMEL'><code>UPPER_CAMEL</code></a> | `UpperCamel`       |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_UNDERSCORE'><code>UPPER_UNDERSCORE</code></a> | `UPPER_UNDERSCORE` |

直接的用法：

```java
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"
```

We find this especially useful, for example, when writing programs that generate other programs.

我们发现特别有用，例如当我们编写代码生成器的时候。