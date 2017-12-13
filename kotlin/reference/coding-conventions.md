# 代码约定



## 命名风格

默认使用 Java 代码约定：

* 使用驼峰命名（避免下划线）
* 类型以大写开头
* 方法和成员变量以小写开头
* 使用 4 空格缩进
* 公有方法应该包含文档



## 冒号

冒号前面用空格分隔当前类型和超类，冒号前面没有空格分隔实例和类型。

``` kotlin
interface Foo<out T : Any> : Bar {
  fun foo(a: Int): T
}
```



## Lambdas

使用 lambdas 表达式，在花括号周围，以及箭头周围，应该使用空格。只要有可能，lambda 应该通过括号外传递。

``` kotlin
list.filter { it > 10 }.map { element -> element * 2 }
```

在短而不嵌套的 lambda 中，建议使用它的约定，而不是显式声明参数。在带参数的嵌套 lambda 中，参数应该总是显式声明。



## 类标题格式

类只有几个参数可以写成一行：

``` kotlin
class Person(id: Int,name: String)
```

具有较长标题的类应该格式化，以便每个主构造函数参数与缩进分开。另外，右括号应该在一个新的行。如果我们使用继承，那么超类的构造函数调用或实现的接口列表应与括号位于同一行：

``` kotlin
class Person(
	id: Int,
  	name: String,
  	surname: String
) : Human(id,name){
  
}
```

对于多个接口，应该首先定位超类的构造函数调用，然后每个接口应该位于不同的行中：

``` kotlin
class Person(
	id: Int,
  	name: String,
  	surname: String
) : Human(id,name),
	KotlinMarker {
      
	}
```

构造函数参数可以使用常规缩进或连续缩进（双倍于常规缩进）。



## Unit

如果方法没有返回值，那么返回类型应该被省略。

``` kotlin
fun foo() {
  
}
```



## 方法 VS 属性

在某些情况下，不带参数的函数可能与只读属性互换。尽管语义是相似的，但是在某种程度上，它们之间存在着某种风格的约定。

当作为基础算法时，以下情况使用属性要优于方法：

* 不会抛出任何异常
* 拥有常数时间复杂度
* 使用成本低（或者第一次运行会缓存）
* 多次调用总是返回相同值