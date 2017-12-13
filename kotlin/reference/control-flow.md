# 控制流



## If 表达式

在 Kotlin 中，`if` 是一个表达式，即它会返回一个值。

``` kotlin
//传统用法
if(a > b){
  max = a
}else {
  max = b
}

//作为表达式
val max = if(a < b) a else b
//代码块
val max = if(a > b) {
  print("Choose a")
  a
}else{
  print("Choose b")
  b
}
```

如果使用 `if` 作为表达式，而不是语句，该表达式需要有 `else` 分支。



## When 表达式

`when` 取代类 C 语言的 `switch` 操作符。

``` kotlin
when (x) {
  1	-> print("x == 1")
  2 -> print("x == 2")
  else -> {
    print("x is neither 1 nor 2")
  }
}
```

如果它被当做表达式， 符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。

如果 `when` 作为表达式使用，则必须有 `else` 分支，除非编译器能够检测出所有可能都已经被覆盖了。

如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：

```kotlin
when (x) {
  0,1	-> print("x== 0 or x == 1")
  else	-> print("otherwise")
}
```

可以使用任意表达式（不只是常量）作为分支条件：

``` kotlin
when (x) {
  parseInt(s)	-> print("s encodes x")
  else			-> print("s does not encode x")
}
```

也可以检测值是否 `in` 或者 `!in` 一个范围或者集合中：

``` kotlin
when (x) {
  in 1..10			-> print("x is in the range")
  in validNumbers	-> print("x is valid")
  !in 10..20		-> print("x is outside the range")
  else				-> print("none of the above")
}
```

另一种可能性是检测一个值 `is` 或者 `!is` 一个特定类型的值。由于智能转换，你可以访问该类型的方法和属性而无需任何额外的检测。

``` kotlin
fun hasPrefix(x: Any) = when(x) {
  is String		-> x.startsWith("prefix")
  else			-> false
}
```

`when` 也可以用来取代 `if`。如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：

``` kotlin
when {
  x.isOdd()		-> print("x is odd")
  x.isEven()	-> print("x is even")
  else			-> print("x is funny")
}
```



## For 循环

for 循环可以对任何提供迭代器（iterator）的对象进行遍历：

``` kotlin
for (item in collection) print(item)
```

循环体可以是一个代码块。

``` kotlin
for (item: Int in ints){
  //...
}
```

for 循环可以遍历任何提供了迭代器的对象，即：

* 有个成员函数或者扩展函数 `iterator()`，它的返回类型：
  * 有个成员函数或者扩展函数 `next()`。
  * 有个成员函数或者扩展函数 `Boolean hasNext()`。

注意：对数组的 for 循环会被编译为并不创建迭代器的基于索引的循环。

如果你想要通过索引遍历一个数组或者一个 list，你可以这么做：

``` kotlin
for (i in array.indices){
  print(array[i])
}
```

注意这种“在区间上遍历”会编译成优化的实现而不会创建额外对象。

或者可以用库函数 `withIndex`：

``` kotlin
for ((index,value) in array.withIndex()){
  println("the element at $index is $value")
}
```



## While 循环

while. 和 do..while 照常使用。

``` kotlin
while (x > 0){
  x--
}

do {
  val y = retrieveData()
} while (y != null) //y 在此处可见
```



## 循环中的 break 和 continue

支持传统的 break 和 continue。