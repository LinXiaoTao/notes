# 委托



## 类委托

[委托模式](https://zh.wikipedia.org/wiki/%E5%A7%94%E6%89%98%E6%A8%A1%E5%BC%8F)已经证明是实现继承的一个很好的替代方式， 而 Kotlin 可以零样板代码地原生支持它。 类 `Derived` 可以继承一个接口 `Base`，并将其所有共有的方法委托给一个指定的对象：

``` kotlin
interface Base {
  fun print()
}

class BaseImpl(val x: Int) : Base {
  override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
  val b = BaseImpl(10)
  Derived(b).print()
}
```

`Derived` 的超类型列表中的 *by*-子句表示 `b` 将会在 `Derived` 中内部存储。 并且编译器将生成转发给 `b` 的所有 `Base` 的方法。

请注意，覆盖会以你所期望的方式工作：编译器会使用你的 `override` 实现取代委托对象中的实现。如果我们为 `Derived` 添加 `override fun print() { print("abc") }`，该程序会输出“abc”而不是“10”。