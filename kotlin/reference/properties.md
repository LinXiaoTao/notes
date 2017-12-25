# 属性和字段



### 声明属性

Kotlin的类可以有属性。 属性可以用关键字 var 声明为可变的，否则使用只读关键字 val。

``` kotlin
class Address {
  var name: String = ...
  var street: String = ...
}
```

要使用一个属性，只要用名称引用它即可，就像 Java 中的字段：

``` kotlin
fun copyAddress(address: Address): Address {
  val result = Address()
  result.name = address.name
  return result
}
```



### Getters 与 Setters

声明一个属性的完整语法是：

``` kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
	[<getter>]
	[<setter>]
```

其初始器（initializer）、getter 和 setter 都是可选的。属性类型如果可以从初始器 （或者从其 getter 返回值，如下文所示）中推断出来，也可以省略。

例如：

``` kotlin
var allByDefault: Int? //错误：需要显式初始化器，隐含默认 getter 和 setter
var initialized = 1//类型 Int、默认 getter 和 setter
```

一个只读属性的语法和一个可变的属性的语法有两方面的不同：

1. 只读属性使用 `val`
2. 只读属性不允许使用 setter

``` kotlin
val simple: Int? //类型 Int、默认 getter、必须在构造函数中初始化
val inferredType = 1 //类型 Int 、默认 getter
```

我们可以编写自定义的访问器，非常像普通函数，刚好在属性声明内部。这里有一个自定义 getter 的例子：

``` kotlin
val isEmpty: Boolean
	get() = this.size == 0
```

一个自定义的 setter 的例子：

``` kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 解析字符串并赋值给其他属性
    }
```

按照惯例，setter 参数的名称是 `value`，但是如果你喜欢你可以选择一个不同的名称。

自 Kotlin 1.1 起，如果可以从 getter 推断出属性类型，则可以省略它：

``` kotlin
val isEmpty get() = this.size == 0 //具有类型 Boolean
```

如果你需要改变一个访问器的可见性或者对其注解，但是不需要改变默认的实现， 你可以定义访问器而不定义其实现：

``` kotlin
var setterVisibility: String = "abc"
	private set

var setterWithAnnotation: Any? = null
	@Inject set
```



### 幕后字段

Kotlin 类不能有字段。然而，当使用自定义访问器时，有时有一个幕后字段（backing field）有时是必要的。为此  Kotlin 提供了一个自动幕后字段，它可通过 `field` 标识符访问。

``` kotlin
var counter = 0
	set(value) {
      if (value >= 0)
      	field = value
	}
```

`field` 标识符只能用在属性的访问器内。

如果属性至少一个访问器使用默认实现，或者自定义访问器通过 `field` 引用幕后字段，将会为该属性生成一个幕后字段。

例如，下面的情况下， 就没有幕后字段：

``` kotlin
val isEmpty: Boolean
	get() = this.size == 0
```



### 幕后属性

如果你的需求不符合这套隐式的幕后字段方案，那么总是可以使用幕后属性；

``` kotlin
private var _table: Map<String,Int>? = null
public val table: Map<String,Int>
	get() {
      if(_table == null){
        _table = HashMap()
      }
      return _table ?: throw AssertionError("Set to null by another thread")
	}
```

从各方面看，这正是与 Java 相同的方式。因为通过默认 getter 和 setter 访问私有属性会被优化，所以不会引入函数调用开销。



### 编译器常量

已知值的属性可以使用 `const` 修饰符标记为编译期常量。这些属性需要满足以下要求：

* 位于顶层或者是 `object` 的成员
* 用 `String` 或原生类型值初始化
* 没有自定义 getter

这些属性可以用在注解：

``` kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() {}
```



### 延迟初始化属性与变量

一般地，属性声明为非空类型必须在构造函数中初始化。 然而，这经常不方便。例如：属性可以通过依赖注入来初始化， 或者在单元测试的 setup 方法中初始化。 这种情况下，你不能在构造函数内提供一个非空初始器。 但你仍然想在类体中引用该属性时避免空检查。

为处理这种情况，你可以用 `lateinit` 修饰符标记该属性：

``` kotlin
public class MyTest {
  lateinit var subject: TestSubject
  @Setup fun setup() {
    subject = TestSubject()
  }
  @Test fun test {
    subject.method()
  }
}
```

该修饰符只能用于在类体中的属性（不是在主构造函数中声明的 `var` 属性，并且仅当该属性没有自定义 getter 或 setter 时），而自 Kotlin 1.2 起，也用于顶层属性与局部变量。该属性或变量必须为非空类型，并且不能是原生类型。

在初始化前访问一个 `lateinit` 属性会抛出一个特定异常，该异常明确标识该属性被访问及它没有初始化的事实。



### 检测一个 lateinit var 是否已初始化（自 1.2 起）

要检测一个 `lateinit var` 是否已经初始化过，在该属性的引用上使用 `.isInitialized`：

``` kotlin
if ( foo::bar.isInitialized) {
  println(foo.bar)
}
```

此检测仅对可词法级访问的属性可用，即声明位于同一个类型内、位于其中一个外围类型中或者位于相同文件的顶层的属性。



### 覆盖属性



### 委托属性

最常见的一类属性就是简单地从幕后字段中读取（以及可能的写入）。 另一方面，使用自定义 getter 和 setter 可以实现属性的任何行为。 介于两者之间，属性如何工作有一些常见的模式。一些例子：惰性值、 通过键值从映射读取、访问数据库、访问时通知侦听器等等。