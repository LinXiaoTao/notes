# 基础类型

在 Kotlin，一切都是对象。有些类型会有特定内部表示，比如，numbers，characters 和 booleans 在运行时可以用基本类型表示，但对于使用者来说，它们的行为就是一个对象。



## Numbers

Kotlin 处理 numbers 和 Java 相似，不过不完全相同。比如，对于 numbers 没有隐式转换，并且 literals 在某些情况下略有不同。

> literals：literal 在源码中是一个表示固定值的符号。
>
> ``` kotlin
> val a = 1		// 1 是 integer 类型的 literal
> val s = "cat"	// "cat" 是 string 类型的 literal
> ```
>
> 对象的 literal 类似匿名类。

Kotlin 提供以下内置类型表示 numbers：

| Type   | Bit width |
| ------ | --------- |
| Double | 64        |
| Float  | 32        |
| Long   | 64        |
| Int    | 32        |
| Short  | 16        |
| Byte   | 8         |

记住 characters 在 Kotlin 中不属于 numbers。



### 常量 Literal

允许以下 integral 常量 literal：

* 十进制：`123`
  * 如果是 Long：`123L`
* 十六进制：`0x0F`
* 二进制：`0b0001011`

不支持八进制。

Kotlin 也支持传统的浮点 numbers：

* 默认双精度浮点类型：`123.5`，`123.5e10`
* 单精度浮点类型：`123.5f`，`123.5F`



### numbers literals 中的下划线（从 1.1 以后）

可以使用下划线使数字常量更具可读性：

``` kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
```



## 表示方式

在 Java 上，numbers 在物理上被作为 JVM 基本类型保存，除非我们需要可空引用或者用于泛型，那么我们就需要用到它们的包装类。

注意数字装箱不必保留同一性：

``` kotlin
val a: Int = 1000
print(a === a)//true
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === annotherBoxedA)//false!!!
```

``` kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

> 使用 === 会比较 numbers 的包装类引用，使用 == 比较基本类型的值。



### 显式转换

因此，较小的类型不会被隐式转换为较大的类型，必须显式进行转换。

``` kotlin
val b: Byte = 1
val i: Int = b //ERROR
val i: Int = b.toInt()//OK
```

缺乏隐式类型转换并不显著，因为类型会从上下文推断出来，而算术运算会有重载做适当转换，例如：

``` kotlin
val l = 1L + 3 // Long + Int => Long
```



### 运算

Kotlin 支持对数字进行标准的算术运算，这些运算被声明为适当的类的成员（但是编译器将调用优化到相应的指令）。

``` kotlin
val x = (1 shl 2) and 0x000FF000
```



### 浮点类型的比较

当其中的操作数  `a`  与  `b`  都是静态已知的  `Float`  或  `Double`  或者它们对应的可空类型（声明为该类型，或者推断为该类型，或者[智能类型转换](https://www.kotlincn.net/docs/reference/typecasts.html#智能转换)的结果是该类型），两数字所形成的操作或者区间遵循 IEEE 754 浮点运算标准。

当比较操作的是非静态  `Float` 或者 `Double`（比如：泛型，`Any`）等等，这些操作使用为 `Float` 与 `Double` 实现的不符合标准的  `equals` 与 `compareTo`，这会出现：

* `NaN` 被认为等于自身
* `NaN` 被大于其他所有元素，包括 `POSITIVE_INFINITY`
* `-0.0` 被认为小于 `0.0`



### 字符

`Characters` 被用来表示 `Char`。它们不可以被直接作为 numbers。

``` kotlin
fun check(c: Char){
  if(c == 1){//ERROR
    
  }
}
```

字符的 literals 用 单引号表示 `'1'`。`Characters` 支持使用转义字符：`\t`，`\b`，`\n`，`\'`，`\"`，`\\`，`\$`。如果要转义其他字符，使用 Unicode 转义字符语法：`\uFF00`。

我们可以明确将字符转换为 `Int` number：

``` kotlin
fun decimalDigitValue(c: Char): Int{
  if(c !in '0'..'9')
  	throw IllegalArgumentException("Out of range")
  return c.toInt() - '0'.toInt()
}
```



### 布尔值

使用 `Boolean` 表示布尔值。

布尔值的内置操作符包括：

* `II` 或比较
* `&&` 且比较
* `!` 不等于



### 数组

在 Kotlin 中数组使用 `Array` 表示。拥有 `get` 和 `set` 方法（通过运算符重载变成 `[]`），并且还有 `size` 属性，还有其他有用的成员方法。

``` kotlin
class Array<T> private consturctor() {
  val size: Int
  operator fun get(index: Int): T
  operator fun set(index: Int,value: T): Unit
}
```

可以通过使用方法 `arrayOf()`，并且传递值给它，比如：`arrayOf(1,2,3)` 将创建数组 [1,2,3]。另外还可以通过，`arrayOfNulls()` 创建给定大小的空元素数组。

可以使用 `Array` 的构造方法，传递数组大小，并且初始化数组每个元素的值。

``` kotlin
val asc = Array(5,{ i -> (i * i).toString() })
```

注意：Kotlin 不会让我们将 `Array<String` 分配给 `Array<Any>` ，这将防止运行时错误（不过你可以使用 `Array<out Any>` ）。

Kotlin 提供了特定的数组去表示基本类型的数组，而不是使用包装类型：`ByteArray`，`ShortArray`，`IntArray`。这些类与 `Array` 没有继承关系，不过它们拥有相同的方法和属性。

``` kotlin
val x: IntArray = intArrayOf(1,2,3)
x[0] = x[1] + x[2]
```



### 字符串

字符串用 `String` 类型表示。字符串是不可变的。 字符串的元素——字符可以使用索引运算符访问: `s[i]`。 可以用 for 循环迭代字符串：

``` kotlin
for (c in str){
  println(c)
}
```



### 字符串 literals

Kotlin 有两种类型的字符串 literals：转义字符串可以有转义字符，以及原生字符串可以包含换行和任意文本。

转义字符串很像  Java 字符串：

``` kotlin
val s = "Hello, world!\n"
```

转义采用传统的反斜杠方式。参见上面支持的转义序列。

原生字符串使用三个引号 `"""` 包裹，内部没有转义，并且可以包含换行和任何其他字符：

``` kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```



### 字符串模版

字符串可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（`$`）开头，由一个简单的名字构成：

``` kotlin
val i = 10
val s = "i = $i" // 求值结果为 "i = 10"
```

或者用花括号括起来的任意表达式:

``` kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // 求值结果为 "abc.length is 3"
```

原生字符串和转义字符串内部都支持模板。 如果你需要在原生字符串中表示字面值 `$` 字符（它不支持反斜杠转义），你可以用下列语法：

``` kotlin
val price = """
${'$'}9.99
"""
```

