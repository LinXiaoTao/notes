# 返回和跳转

Kotlin 有三种结构化跳转表达式：

* return。默认从最直接包围它的函数或者匿名函数返回。
* break。终止最直接包围它的循环。
* continue。继续下一次最直接包围它的循环。

所有这些表达式都可以用作更大表达式的一部分：

``` kotlin
val s = person.name ?: return
```

这些表达式的类型是 `Nothing`。



## Break 与 Continue 标签

在 Kotlin 中任何表达式都可以用标签 `label` 来标记。标签的格式为标识符后跟 `@` 符号，例如：`abc@`、`fooBar@` 都是有效的标签。

``` kotlin
loop@ for (i in 1..10){
  
}
```

现在，我们可以用标签限制 break 或者 continue：

``` kotlin
loop@ for (i in 1..100){
  for (j in 1..100){
    if(...) break@loop
  }
}
```



## 标签处返回

Kotlin 有函数 literals，局部函数和对象表达式。因此 Kotlin 的函数可以被嵌套。标签限制的 return 允许我们从外层函数返回。 最重要的一个用途就是从 lambda 表达式中返回。

``` kotlin
fun foo() {
  ints.forEach {
    if(it == 0) return
    print(it)
  }
}
```

这个 return 表达式从最直接包围它的函数即 `foo` 中返回。（注意，这种非局部的返回只支持传给内联函数的 lambda 表达式。）如果我们需要从 lambda 表达式中返回，我们必须给它加标签并用以限制 return。

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。 该标签与接受该 lambda 的函数同名。

``` kotlin
fun foo() {
  ints.forEach lit@ {
    if (it == 0) return@lit
    print(it)
  }
}
```

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。 该标签与接受该 lambda 的函数同名。

``` kotlin
fun foo() {
  ints.forEach {
    if (it == 0) return@forEach
    print(it)
  }
}
```

或者，我们用一个匿名函数替代 lambda 表达式。 匿名函数内部的 return 语句将从该匿名函数自身返回

``` kotlin
fun foo() {
  ints.forEach(fun(value: Int){
    if(value == 0) return
    print(value)
  })
}
```

当要返一个回值的时候，解析器优先选用标签限制的 return，即

``` kotlin
return@a 1
```

