# 泛型

与 Java 类似，Kotlin 中的类也可以有类型参数：

``` kotlin
class Bax<T>(t: T){
  var value = t
}
```

一般来说，要创建这样类的实例，我们需要提供类型参数：

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

但是如果类型参数可以推断出来，例如从构造函数的参数或者从其他途径，允许省略类型参数：

``` kotlin
val box = Box(1)
```



## 型变

Java 类型系统中最棘手的部分之一是通配符类型（参见 [Java Generics FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)）。 而 Kotlin 中没有。 相反，它有两个其他的东西：声明处型变（declaration-site variance）与类型投影（type projections）。



### 声明处型变

在 Kotlin 中，有一种方法向编译器解释这种情况。这称为**声明处型变**：我们可以标注 `Source` 的**类型参数** `T` 来确保它仅从 `Source<T>` 成员中**返回**（生产），并从不被消费。 为此，我们提供 **out** 修饰符：

``` kotlin
abstract class Source<out T> {
  abstract fun nextT(): T
}

fun demo(strs: Source<String>){
  val objects: Source<Any> = strs
}
```

一般原则是：当一个类 `C` 的类型参数 `T` 被声明为 **out** 时，它就只能出现在 `C` 的成员的**输出**-位置，但回报是 `C<Base>` 可以安全地作为 `C<Derived>`的超类。

简而言之，他们说类 `C` 是在参数 `T` 上是**协变的**，或者说 `T` 是一个**协变的**类型参数。 你可以认为 `C` 是 `T` 的**生产者**，而不是 `T` 的**消费者**。

**out **修饰符称为**型变注解**，并且由于它在类型参数声明处提供，所以我们讲**声明处型变**。 这与 Java 的**使用处型变**相反，其类型用途通配符使得类型协变。

另外除了 **out**，Kotlin 又补充了一个型变注释：**in**。它使得一个类型参数**逆变**：只可以被消费而不可以被生产。逆变类的一个很好的例子是 `Comparable`：

```kotlin
abstract class Comparable<in T> {
  abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
 	x.compareTo(1.0)//1.0 拥有类型 Double，它是 Number 的子类型
  	val y: Comparable<Double> = x//因此，我们可以将 x 赋给类型为 Comparable <Double> 的变量
}
```



## 类型投影

### 使用处型变：类型投影

将类型参数 T 声明为 *out* 非常方便，并且能避免使用处子类型化的麻烦，但是有些类实际上**不能**限制为只返回 `T`！ 一个很好的例子是 Array：

``` kotlin
class Array<T>(val size: Int) {
  fun get(index: Int): T {}
  fun set(index: Int,value: T) {}
}
```

该类在 `T` 上既不能是协变的也不能是逆变的。这造成了一些不灵活性。考虑下述函数：

``` kotlin
fun copy(from: Array<Any>,to: Array<Any) {
  assert(from.size == to.size)
  for (i in from.indices)
  	to[i] = from[i]
}
```

这个函数应该将项目从一个数组复制到另一个数组。让我们尝试在实践中应用它：

``` kotlin
val ints: Array<Int> = arrayOf(1,2,3)
val any: Array<Any>(3) { "" }
copy(ints,any) //error
```

这里我们遇到同样熟悉的问题：`Array <T>` 在 `T` 上是**不型变的**，因此 `Array <Int>` 和 `Array <Any>` 都不是另一个的子类型。为什么？ 再次重复，因为 copy **可能**做坏事，也就是说，例如它可能尝试**写**一个 String 到 `from`， 并且如果我们实际上传递一个 `Int` 的数组，一段时间后将会抛出一个 `ClassCastException` 异常。

那么，我们唯一要确保的是 `copy()` 不会做任何坏事。我们想阻止它**写**到 `from`，我们可以：

``` kotlin
fun copy(from: Array<out Any),to: Array<Any>) {
  
}
```

里发生的事情称为**类型投影**：我们说`from`不仅仅是一个数组，而是一个受限制的（**投影的**）数组：我们只可以调用返回类型为类型参数 `T` 的方法，如上，这意味着我们只能调用 `get()`。这就是我们的**使用处型变**的用法，并且是对应于 Java 的 `Array<? extends Object>`、 但使用更简单些的方式。

你也可以使用 **in** 投影一个类型：

``` kotlin
fun fill(dest: Array<in String>,value: String) {
  
}
```

`Array<in String>` 对应于 Java 的 `Array<? super String>`，也就是说，你可以传递一个 `CharSequence` 数组或一个 `Object` 数组给 `fill()` 函数。



### 星投影

有时你想说，你对类型参数一无所知，但仍然希望以安全的方式使用它。 这里的安全方式是定义泛型类型的这种投影，该泛型类型的每个具体实例化将是该投影的子类型。

Kotlin 为此提供了所谓的**星投影**语法：

* 对于 `Foo <out T>`，其中 `T` 是一个具有上界 `TUpper` 的协变类型参数，`Foo <*>` 等价于 `Foo <out TUpper>`。 这意味着当 `T` 未知时，你可以安全地从 `Foo <*>` *读取* `TUpper` 的值。
* 对于 `Foo <in T>`，其中 `T` 是一个逆变类型参数，`Foo <*>` 等价于 `Foo <in Nothing>`。 这意味着当 `T` 未知时，没有什么可以以安全的方式*写入* `Foo <*>`。
* 对于 `Foo <T>`，其中 `T` 是一个具有上界 `TUpper` 的不型变类型参数，`Foo<*>` 对于读取值时等价于 `Foo<out TUpper>` 而对于写值时等价于 `Foo<in Nothing>`。

如果泛型类型具有多个类型参数，则每个类型参数都可以单独投影。 例如，如果类型被声明为 `interface Function <in T, out U>`，我们可以想象以下星投影：

* `Function<*, String>` 表示 `Function<in Nothing, String>`；
* `Function<Int, *>` 表示 `Function<Int, out Any?>`；
* `Function<*, *>` 表示 `Function<in Nothing, out Any?>`。

*注意*：星投影非常像 Java 的原始类型，但是安全。



## 泛型函数

不仅类可以有类型参数。函数也可以有。类型参数要放在函数名称之前：

``` kotlin
fun <T> singletonList(item: T): List<T> {
  
}

fun <T> T.basicToString() : String {
  
}
```

要调用泛型函数，在调用处函数名**之后**指定类型参数即可：

``` kotlin
val l = singletonList<Int>(1)
```



### 泛型约束

能够替换给定类型参数的所有可能类型的集合可以由**泛型约束**限制。



### 上届

最常见的约束类型是与 Java 的 *extends* 关键字对应的 **上界**：

``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
  
}
```

冒号之后指定的类型是**上界**：只有 `Comparable<T>` 的子类型可以替代 `T`。 例如：

```  kotlin
sort(listOf(1,2,3)) //ok
sort(listOf(HashMap<Int,String>())) //error
```

默认的上界（如果没有声明）是 `Any?`。在尖括号中只能指定一个上界。 如果同一类型参数需要多个上界，我们需要一个单独的 **where**-子句：

``` kotlin
fun <T> cloneWhenGreater(list: List<T>,threshold: T) : List<T>
	where T : Comparable<T>,
		  T : Cloneable {
            return list.filter { it > threshold }.map { it.clone() }
		  }
```

