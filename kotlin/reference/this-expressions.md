# This 表达式



为了表示当前的 *接收者* 我们使用 *this* 表达式：

- 在[类](https://www.kotlincn.net/docs/reference/classes.html#%E7%BB%A7%E6%89%BF)的成员中，*this* 指的是该类的当前对象。
- 在[扩展函数](https://www.kotlincn.net/docs/reference/extensions.html)或者[带接收者的函数字面值](https://www.kotlincn.net/docs/reference/lambdas.html#%E5%B8%A6%E6%8E%A5%E6%94%B6%E8%80%85%E7%9A%84%E5%87%BD%E6%95%B0%E5%AD%97%E9%9D%A2%E5%80%BC)中， *this* 表示在点左侧传递的 *接收者* 参数。

如果 *this* 没有限定符，它指的是最内层的包含它的作用域。要引用其他作用域中的 *this*，请使用 *标签限定符*：



### 限定的 this

要访问来自外部作用域的*this*（一个[类](https://www.kotlincn.net/docs/reference/classes.html) 或者[扩展函数](https://www.kotlincn.net/docs/reference/extensions.html)， 或者带标签的[带接收者的函数字面值](https://www.kotlincn.net/docs/reference/lambdas.html#%E5%B8%A6%E6%8E%A5%E6%94%B6%E8%80%85%E7%9A%84%E5%87%BD%E6%95%B0%E5%AD%97%E9%9D%A2%E5%80%BC)）我们使用`this@label`，其中 `@label` 是一个代指 *this* 来源的标签：

``` kotlin
class A { //标签 @A
  inner class B { //标签 @B
    fun Int.foo() { //标签 @foo
      val a = this@A // A 的 this
      val b = this@B // B 的 this
      
      val c = this //foo() 的接收者
      val c1 = this@foo // foo() 的接收者
      
      val funList = lambda@ fun String.() {
        val d = this // funList 的接收者
      }
      
      val funList2 = { s: String ->
        val d1 = this // foo() 的接收者，因为它包含 lambda 表达式没有任何接收者
      }
    }
  }
}
```

