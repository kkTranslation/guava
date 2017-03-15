# Example
```java

int logFloor = LongMath.log2(n, FLOOR);

int mustNotOverflow = IntMath.checkedMultiply(x, y);

long quotient = LongMath.divide(knownMultipleOfThree, 3, RoundingMode.UNNECESSARY); // fail fast on non-multiple of 3

BigInteger nearestInteger = DoubleMath.roundToBigInteger(d, RoundingMode.HALF_EVEN);

BigInteger sideLength = BigIntegerMath.sqrt(area, CEILING);
```

# Why?
* Guava Math 已经针对异常溢出情况进行了全面测试。对溢出语义，Guava 在相关文档中有明确的说明；如果运算的溢出检查不能通过，将导致快速失败；
* Guava Math 经过精心的基准测试和优化。 虽然性能不可避免地取决于特定的硬件细节，它们的速度与Apache Commons `MathUtils`中的类似功能相比，并且在某些情况下显著更好。
    * Guava Math 在设计上考虑了可读性和正确的编程习惯。 `IntMath.log2(x，CEILING)`的意义是明确和明显的，即使在随意的通读。 `32 - Integer.numberOfLeadingZeros(x - 1)`的意思不是。
    
注意：Guava Math 与GWT不是特别兼容，也没有针对GWT进行优化，因为不同的溢出逻辑。

# 整数运算
Guava Math 主要处理三种整数类型：`int`，`long`和`BigInteger`。 这些运算工具方便地命名为[IntMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html), [LongMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html) 和 [BigIntegerMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html)。

## 检查运算

Guava为`IntMath`和`LongMath`提供算术方法，在溢出时快速失败，而不是默默忽略它。

| `IntMath`                                | `LongMath`                               |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedAdd(int, int)'><code>IntMath.checkedAdd</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedAdd(long, long)'><code>LongMath.checkedAdd</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedSubtract(int, int)'><code>IntMath.checkedSubtract</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedSubtract(long, long)'><code>LongMath.checkedSubtract</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedMultiply(int, int)'><code>IntMath.checkedMultiply</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedMultiply(long, long)'><code>LongMath.checkedMultiply</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedPow(int, int)'><code>IntMath.checkedPow</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedPow(long, long)'><code>LongMath.checkedPow</code></a> |

```java

IntMath.checkedAdd(Integer.MAX_VALUE, Integer.MAX_VALUE); // throws ArithmeticException
```

# 实数运算

`IntMath`，`LongMath`和`BigIntegerMath`支持具有“精确实值”的各种方法，但将其结果四舍五入为整数。 这些方法接受[java.math.RoundingMode](http://docs.oracle.com/javase/7/docs/api/java/math/RoundingMode.html)。 这与JDK中使用的`RoundingMode`相同，并且是具有以下值的枚举：
* `DOWN`: 向0舍入（这是Java格式的特性。）
* `UP`: 从0开始。
    * `FLOOR`: 向负无限大方向舍入
    * `CEILING`: 向正无限大方向舍入
    * `UNNECESSARY`: 不需要舍入，如果用此模式进行舍入，应直接抛出 `ArithmeticException`
    * `HALF_UP`:向最近的整数舍入，其中 x.5 远离零方向舍入
    * `HALF_DOWN`: 向最近的整数舍入，其中 x.5 向零方向舍入
    * `HALF_EVEN`: 向最近的整数舍入，其中 x.5 向相邻的偶数舍入

这些方法旨在提高代码的可读性，例如，`divide(x, 3, CEILING)` 即使在快速阅读时也是清晰。

另外，这些函数中的每一个在内部只使用整数运算，除了在构造用于`sqrt`的初始近似值之外。

| 运算         | `IntMath`                                | `LongMath`                               | `BigIntegerMath`                         |
| :---------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| 除法          | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#divide(int, int, java.math.RoundingMode)'><code>divide(int, int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#divide(long, long, java.math.RoundingMode)'><code>divide(long, long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#divide(java.math.BigInteger, java.math.BigInteger, java.math.RoundingMode)'><code>divide(BigInteger, BigInteger, RoundingMode)</code></a> |
| 2为底的对数  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log2(int, java.math.RoundingMode)'><code>log2(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log2(long, java.math.RoundingMode)'><code>log2(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log2(java.math.BigInteger, java.math.RoundingMode)'><code>log2(BigInteger, RoundingMode)</code></a> |
| 10为底的对数 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log10(int, java.math.RoundingMode)'><code>log10(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log10(long, java.math.RoundingMode)'><code>log10(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log10(java.math.BigInteger, java.math.RoundingMode)'><code>log10(BigInteger, RoundingMode)</code></a> |
| 平方根       | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#sqrt(int, java.math.RoundingMode)'><code>sqrt(int, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#sqrt(long, java.math.RoundingMode)'><code>sqrt(long, RoundingMode)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#sqrt(java.math.BigInteger, java.math.RoundingMode)'><code>sqrt(BigInteger, RoundingMode)</code></a> |

```java
BigIntegerMath.sqrt(BigInteger.TEN.pow(99), RoundingMode.HALF_EVEN);
   // returns 31622776601683793319988935444327185337195551393252
```

## 附加功能
Guava 还另外提供了一些其他有用的运算函数

| 运算                                | `IntMath`                                | `LongMath`                               | `BigIntegerMath`                         |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| 最大公约数                 | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#gcd(int, int)'><code>gcd(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#gcd(long, long)'><code>gcd(long, long)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#gcd(java.math.BigInteger)'><code>BigInteger.gcd(BigInteger)</code></a> |
| 取模（总是非负的，-5模3是1） | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#mod(int, int)'><code>mod(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#mod(long, long)'><code>mod(long, long)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#mod(java.math.BigInteger)'><code>BigInteger.mod(BigInteger)</code></a> |
| 指数（可能溢出）            | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#pow(int, int)'><code>pow(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#pow(long, int)'><code>pow(long, int)</code></a> | In JDK: <a href='http://docs.oracle.com/javase/6/docs/api/java/math/BigInteger.html#pow(int)'><code>BigInteger.pow(int)</code></a> |
| 功率二测试                  | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#isPowerOfTwo(int)'><code>isPowerOfTwo(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#isPowerOfTwo(long)'><code>isPowerOfTwo(long)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#isPowerOfTwo(java.math.BigInteger)'><code>isPowerOfTwo(BigInteger)</code></a> |
| 阶乘（如果输入太大则返回“MAX_VALUE”） | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#factorial(int)'><code>factorial(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#factorial(int)'><code>factorial(int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#factorial(int)'><code>factorial(int)</code></a> |
| 二项式系数（如果太大则返回“MAX_VALUE”） | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#binomial(int, int)'><code>binomial(int, int)</code></a> |

# Floating-point arithmetic
Floating point arithmetic is pretty thoroughly covered by the JDK, but we added a few useful methods to [DoubleMath](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html).

| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#isMathematicalInteger(double)'><code>isMathematicalInteger(double)</code></a> | Tests if the input is finite and an exact integer. |
| :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToInt(double, java.math.RoundingMode)'><code>roundToInt(double, RoundingMode)</code></a> | Rounds the specified number and casts it to an int, if it fits into an int, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToLong(double, java.math.RoundingMode)'><code>roundToLong(double, RoundingMode)</code></a> | Rounds the specified number and casts it to a long, if it fits into a long, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToBigInteger(double, java.math.RoundingMode)'><code>roundToBigInteger(double, RoundingMode)</code></a> | Rounds the specified number to a `BigInteger`, if it is finite, failing fast otherwise. |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#log2(double, java.math.RoundingMode)'><code>log2(double, RoundingMode)</code></a> | Takes the base-2 logarithm, and rounds to an `int` using the specified `RoundingMode`.  Faster than `Math.log(double)`. |
