# 反射



反射是这样的一组语言和库功能，它允许在运行时自省你的程序的结构。 Kotlin 让语言中的函数和属性做为一等公民、并对其自省（即在运行时获悉一个名称或者一个属性或函数的类型）与简单地使用函数式或响应式风格紧密相关。

> 在 Java 平台上，使用反射功能所需的运行时组件作为单独的 JAR 文件（`kotlin-reflect.jar`）分发。这样做是为了减少不使用反射功能的应用程序所需的运行时库的大小。如果你需要使用反射，请确保该 .jar文件添加到项目的 classpath 中。



### 类引用

最基本的反射功能是获取 Kotlin 类的运行时引用。要获取对静态已知的 Kotlin 类的引用，可以使用 *类字面值* 语法：

``` kotlin
val c = MyClass::class
```

该引用是 [KClass](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html) 类型的值。

请注意，Kotlin 类引用与 Java 类引用不同。要获得 Java 类引用， 请在 `KClass` 实例上使用 `.java` 属性。



### 绑定的类引用（自 1.1 起）

通过使用对象作为接收者，可以用相同的 `::class` 语法获取指定对象的类的引用：

``` kotlin
val widget: Widget = Widegt()
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

你可以获取对象的精确类的引用，例如 `GoodWidget` 或 `BadWidget`，尽管接收者表达式的类型是 `Widget`。



### 函数引用

当我们有一个命名函数声明如下：

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
```

我们可以很容易地直接调用它（`isOdd(5)`），但是我们也可以把它作为一个值传递。例如传给另一个函数。 为此，我们使用 `::` 操作符：

``` kotlin
val numbers = listOf(1,2,3)
println(numbers.filter(::isOdd))
```

这里 `::isOdd` 是函数类型 `(Int) -> Boolean` 的一个值。

当上下文中已知函数期望的类型时，`::` 可以用于重载函数。 例如：

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

val numbers = listOf(1,2,3)
println(numbers.filter(::isOdd)) //引用到 isOdd(x: Int)
```

或者，你可以通过将方法引用存储在具有显式指定类型的变量中来提供必要的上下文：

``` kotlin
val predicate: (String) -> Boolean = ::isOdd //引用到 isOdd(x: String)
```

如果我们需要使用类的成员函数或扩展函数，它需要是限定的。 例如 `String::toCharArray` 为类型 `String`提供了一个扩展函数：`String.() -> CharArray`。



### 示例：函数组合

考虑以下函数：

``` kotlin
fun <A,B,C> compose(f: (B) -> C,g: (A) -> B): (A) -> C {
  return { x -> f(g(x))}
}
```

它返回一个传给它的两个函数的组合：`compose(f, g) = f(g(*))`。 现在，你可以将其应用于可调用引用：

``` kotlin
fun length(s: String) = s.length

val oddLength = compose(::isOdd,::length)

val strings = listOf("a","b","abc")

println(strings.filter(oddLength))

```



### 属性引用

要把属性作为 Kotlin中 的一等对象来访问，我们也可以使用 `::` 运算符：

``` kotlin
val x = 1

fun main(args: Array<String>) {
  println(::x.get())
  prinktln(::x.name)
}
```

表达式 `::x` 求值为 `KProperty<Int>` 类型的属性对象，它允许我们使用 `get()` 读取它的值，或者使用 `name` 属性来获取属性名。更多信息请参见[关于 `KProperty` 类的文档](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)。

对于可变属性，例如 `var y = 1`，`::y` 返回 [`KMutableProperty`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html) 类型的一个值， 该类型有一个 `set()` 方法。

``` kotlin
val strs = listOf("a","bc","def")

fun main(args: Array<String>) {
  ::y.set(2)
  println(y)
}
```

属性引用可以用在不需要参数的函数处：

> 在 kotlin-reflect 1.1.4-3 上下面的代码会报错：`KotlinReflectionInternalError`

``` kotlin
val strs = listOf("a","bc","def")
println(strs.map(String::length))
```

要访问属于类的成员的属性，我们这样限定它：

``` kotlin

fun main(args: Array<String>) {
  val prop = A::p
  println(prop.get(A(1)))
}

data class A(val p: Int)
```

对于扩展属性：

``` kotlin
val String.lastChar: Char
	get() = this[length - 1]

fun main(args: Array<String>) {
  println(String::lastChar.get("abc"))
}
```



### 与 Java 反射的互操作性

在Java平台上，标准库包含反射类的扩展，它提供了与 Java 反射对象之间映射（参见 `kotlin.reflect.jvm`包）。 例如，要查找一个用作 Kotlin 属性 getter 的 幕后字段或 Java方法，可以这样写：

``` kotlin
import kotlin.reflect.jvm.*

class A(val p: Int)

fun main(args: Array<String>) {
  println(A::p.javaGetter)
  println(A::p.javaField)
}
```

要获得对应于 Java 类的 Kotlin 类，请使用 `.kotlin` 扩展属性：

```kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```



### 构造函数的引用

构造函数可以像方法和属性那样引用。他们可以用于期待这样的函数类型对象的任何地方：它与该构造函数接受相同参数并且返回相应类型的对象。 通过使用 `::` 操作符并添加类名来引用构造函数。考虑下面的函数， 它期待一个无参并返回 `Foo` 类型的函数参数：

``` kotlin
class Foo

fun function(factory: () -> Foo) {
  val x: Foo = factory()
}
```

使用 `::Foo`，类 Foo 的零参数构造函数，我们可以这样简单地调用它：

``` kotlin
function(:Foo)
```



### 绑定的函数与属性引用（自 1.1 起）

你可以引用特定对象的实例方法：

``` kotlin
val numberRegex = "\\d+".toRegex()
println(numberRegex.matches("29"))

val isNumber = numberRegex::matches
println(isNumber("29"))
```

取代直接调用方法 `matches` 的是我们存储其引用。 这样的引用会绑定到其接收者上。 它可以直接调用（如上例所示）或者用于任何期待一个函数类型表达式的时候：

``` kotlin
val strings = listOf("abc","124","a70")
println(strings.filter(numberRegex::matches))
```

比较绑定的类型和相应的未绑定类型的引用。 绑定的可调用引用有其接收者“附加”到其上，因此接收者的类型不再是参数：

``` kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches
val matches: (Regex,CharSequence) -> Boolean = Regex::matches
```

属性引用也可以绑定：

``` kotlin
val prop = "abc"::length
println(prop.get())
```

自 Kotlin 1.2 起，无需显式指定 `this` 作为接收者：`this::foo` 与 `::foo` 是等价的。