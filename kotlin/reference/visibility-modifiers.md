# 可见性修饰符

类、对象、接口、构造函数、方法、属性和它们的 setter 都可以有可见性修饰符。（ getter 总是与属性有着相同的可见性。）在 Kotlin 中有这四个可见性修饰符：`private`、`protected`、`internal` 和 `public`。默认为 `public`。



### 包

函数、属性和类、对象和接口可以在顶层声明，即直接在包内：

``` kotlin
package foo

fun baz() {}
class Bar {}
```

* 如果你不指定任何可见性修饰符，默认为 `public`。
* 如果你声明为 `private`，它只会在声明它的文件内可见。
* 如果你声明为 `internal`，它会在相同模块内可见。
* `protected` 不适用于顶层声明。

注意：要使用另一包中可见的顶层声明，仍需将其导入进来。

``` kotlin
//文件名：example.kt
package foo

private fun foo() {} //在 example.kt 内可见

public var ba: Int = 5 //公开
	private set			//setter 只在 example.kt 内可见

internal val baz = 6 	//相同模块可见

```



### 类和接口

对于类内部声明的成员：

* `private` ：意味着只在这个类内部（包含其所有成员）可见；
* `protect` ：类似 `private` + 只对子类可见；
* `internal`：能见到类声明的本模块内的所有客户端都可见 `internal` 成员；
* `public`：能见到类声明的任何客户端都可见其 `public` 成员；

> 对于 Java 用户，Kotlin 外部类不能访问内部类的 private 成员。

如果覆盖一个 `protected` 成员，并且没有显式指定其可见性，该成员还会是 `protected` 可见性。

``` kotlin
open class Outer {
  private val a = 1
  protected open val b = 2
  internal val c = 3
  val d = 4 //默认 public
  
  protected class Nested {
    public val e: Int = 5
  }
  
  class Subclass : Outer() {
    // a 不可见
    // b,c,d 可见
    //Nested 和 e 可见
    override val b = 5 //protected
  }
  
  class Unrelated(o: Outer) {
    //o.a o.b 不可见
    //o.c 和 o.d 可见，相同模块
    //Outer.Nested 不可见，Nested::e 也不可见
  }
}
```



### 构造函数

要指定一个类的主构造函数的可见性，使用以下语法（需要添加显式的 consstructor 关键字）：

``` kotlin
class C private constructor(a: Int) {...}
```

默认情况下，所有构造函数都是 `public`。



### 局部声明

局部变量，函数和类不能有可见性修饰符



### 模块

可见性修饰符 `internal` 意味着该成员只在相同模块内可见。更具体地说，一个模块是编译在一起的一套 Kotlin 文件：

* 一个 intellij IDEA 模块
* 一个 Maven 项目
* 一个 Gradle 源集
* 一次 <kotlinc> Ant 任务执行所编译的一套文件