# 区间



区间表达式由具有操作符形式 `..` 的 `rangeTo` 函数辅以 *in* 和 *!in* 形成。 区间是为任何可比较类型定义的，但对于整型原生类型，它有一个优化的实现。以下是使用区间的一些示例：

``` kotlin
if (i in 1..10) {
  println(i)
}
```

整型区间（`IntRange`、 `LongRange`、 `CharRange`）有一个额外的特性：它们可以迭代。 编译器负责将其转换为类似 Java 的基于索引的 *for*-循环而无额外开销：

``` kotlin
for (i in 1..4) print(i)
for (i in 4..1) print(i) //none log
```

如果你想倒序迭代数字呢？也很简单。你可以使用标准库中定义的 `downTo()` 函数：

``` kotlin
for (i in 4 downTo 1) print(i)
```

能否以不等于 1 的任意步长迭代数字？ 当然没问题， `step()` 函数有助于此：

``` kotlin
for (i in 1..4 step 2) print(i)

for(i in 4 downTo 1 step 2) print(i)
```

要创建一个不包括其结束元素的区间，可以使用 `until` 函数：

``` kotlin
for (i in 1 until 10) {
  println(i)
}
```



### 它是如何工作的

区间实现了该库中的一个公共接口：`ClosedRange<T>`。

整型数列（`IntProgression`、 `LongProgression`、 `CharProgression`）表示等差数列。 数列由 `first` 元素、`last` 元素和非零的 `step` 定义。 第一个元素是 `first`，后续元素是前一个元素加上 `step`。 `last` 元素总会被迭代命中，除非该数列是空的。

数列是 `Iterable<N>` 的子类型，其中 `N` 分别为 `Int`、 `Long` 或者 `Char`，所以它可用于 *for*-循环以及像 `map`、`filter` 等函数中。 对 `Progression` 迭代相当于 Java/JavaScript 的基于索引的 *for*-循环：

``` kotlin
for(init i = first; i != last; i += step) {
  
}
```

对于整型类型，`..` 操作符创建一个同时实现 `ClosedRange<T>` 和 `*Progression` 的对象。 例如，`IntRange` 实现了 `ClosedRange<Int>` 并扩展自 `IntProgression`，因此为 `IntProgression` 定义的所有操作也可用于 `IntRange`。 `downTo()` 和 `step()` 函数的结果总是一个 `*Progression`。

数列由在其伴生对象中定义的 `fromClosedRange` 函数构造：

``` kotlin
companion object {
  IntProgression.fromClosedRange(start, end, step)
}
```

数列的 `last` 元素这样计算：对于正的 `step` 找到不大于 `end` 值的最大值、或者对于负的 `step` 找到不小于 `end` 值的最小值，使得 `(last - first) % increment == 0`。



## 一些实用函数



### rangeTo()

整型类型的 `rangeTo()` 操作符只是调用 `*Range` 类的构造函数，例如：

```kotlin
class Int {
  operator fun rangeTo(other: Long): LongRange = LongRange(this,other)
  operator fun rangeTo(other: Int): IntRange = IntRange(this,ohter)
}
```

浮点数（`Double`、 `Float`）未定义它们的 `rangeTo` 操作符，而使用标准库提供的泛型 `Comparable` 类型的操作符：

``` kotlin
public operator fun <T: Comparable<T>> T.rangeTo(that: T): CloseRange<T>
```

该函数返回的区间不能用于迭代。



### downTo()

扩展函数 `downTo()` 是为任何整型类型对定义的，这里有两个例子：

``` kotlin
fun Long.downTo(other: Int): LongProgression {
  return LongProgression.fromClosedRange(this,other.toLong(),-1L)
}

fun Byte.downTo(other: Int): IntProgression {
  return IntProgression.fromClosedRange(this.toInt(),other,-1)
}
```



### reversed()

扩展函数 `reversed()` 是为每个 `*Progression` 类定义的，并且所有这些函数返回反转后的数列：

``` kotlin
fun IntProgression.reversed(): IntProgression {
  return InProgression.fromClosedRange(last,first,-step)
}
```



### step()

扩展函数 `step()` 是为每个 `*Progression` 类定义的， 所有这些函数都返回带有修改了 `step` 值（函数参数）的数列。 步长（step）值必须始终为正，因此该函数不会更改迭代的方向：

``` kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

请注意，返回数列的 `last` 值可能与原始数列的 `last` 值不同，以便保持不变式 `(last - first) % step == 0` 成立。这里是一个例子：

``` kotlin
(1..12 step 2).last == 11  // 值为 [1, 3, 5, 7, 9, 11] 的数列
(1..12 step 3).last == 10  // 值为 [1, 4, 7, 10] 的数列
(1..12 step 4).last == 9   // 值为 [1, 5, 9] 的数列
```

