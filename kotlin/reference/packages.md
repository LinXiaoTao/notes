# 包

源文件可能以包声明开头：

``` kotlin
package foo.bar

fun baz() {}

class Goo {}
```



## 默认导入

 默认导入到每个 Kotlin 文件中的包：

* [kotlin.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html)
* [kotlin.annotation.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/index.html)
* [kotlin.collections.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html)
* [kotlin.comparisons.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.comparisons/index.html) （自 1.1 起）
* [kotlin.io.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/index.html)
* [kotlin.ranges.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/index.html)
* [kotlin.sequences.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/index.html)
* [kotlin.text.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/index.html)

根据目标平台额外导入的包：

- JVM
  * java.lang.*
  * [kotlin.jvm.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/index.html)
- JS：
  * [kotlin.js.*](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/index.html)



## 导入

除了默认导入之外，每个文件可以包含它自己的导入指令。 导入语法在[语法](https://www.kotlincn.net/docs/reference/grammar.html#import)中讲述。

可以导入一个单独的名字：

``` kotlin
import foo.Bar
```

也可以导入一个作用域的所有内容（包，类，对象等）：

``` kotlin
import foo.*
```

如果出现名字冲突，可以使用 `as` 关键字在本地重命名冲突项来消歧义：

``` kotlin
import foo.Bar
import bar.Bar as bBar
```

关键字 `import` 并不仅限于导入类；也可用它来导入其他声明：

* 顶层函数以及属性。
* 在对象声明中声明的函数和属性。
* 枚举常量。

与 Java 不同，Kotlin 没有单独的 `import static`，所有的声明都用 `import` 导入。



## 顶层声明的可见性

如果顶层声明是 `private` 的，它是声明它的文件所私有。