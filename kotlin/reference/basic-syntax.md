# 基础语法



### 定义包

``` kotlin
package my.demo
```

包名不需要匹配目录，源文件可以任意放置在文件系统中。



### 定义方法

``` kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

方法体为单个表达式，并且可以自动推断返回类型的方法：

``` kotlin
fun sum(a: Int,b: Int) = a + b
```

方法返回没有意义的值：

``` kotlin
fun printSum(a: Int,b: Int) : Unit{
  println("sum of $a and $b is ${a + b}")
}
```

`Unit` 可以被省略：

``` kotlin
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```



### 定义变量

只读的本地变量：

``` kotlin
val a: Int = 1
val b = 2//推断类型
val c: Int//没有初始值，类型是必须的
c = 3
```

可变变量：

``` kotlin
var x = 5
x += 1
```

Top-level 变量：

``` kotlin
val PI = 3.14
var x = 0

fun incrementX(){
  x += 1
}
```



### 注释

和 Java 和 JavaScript 类似。

``` kotlin
// This is an end-of-line comment
/* This is a block comment
   on multiple lines. */
```

和 Java 不相同的是，Kotlin 支持嵌套注释。



### 使用字符串模版

``` kotlin
var a = 1
val s1 = "a is $a"

a = 2
val s2 = "${s1.replace("is","was")},but now is $a"
```



### 使用条件表达式

``` kotlin
fun maxOf(a: Int,b: Int) : Int{
  if(a > b){
    return a
  }else{
    return b
  }
}
```

使用 if 作为表达式：

``` kotlin
fun maxOf(a: Int,b: Int) = if(a > b) a else b
```



### 使用可空的值和检查 null

当值可能为 null 时，必须明确标记为可空。

``` kotlin
fun parseInt(str: String) : Int?{
  //
}
fun printProduct(arg1: String,arg2: String){
  val x = parseInt(arg1)
  val y = parseInt(arg2)
  if(x != null && y != null){
    //使用可空值必须先做空值检查，否则不能通过编译
    println(x * y)
  }
}
```



### 使用类型检查以及自动转换

is 运算符检查一个表达式是否是一个类型的实例。如果检查的是不可变的局部变量或者成员变量，则不需要再进行转换。

``` kotlin
fun getStringLength(obj: Any) : Int?{
  if(obj is String){
    return obj.length
  }
  return null
}
```

或者

``` kotlin
fun getStringLength(obj: Any) : Int?{
  if(obj !is String) return null
  return obj.length
}
```



### 使用 for 循环

``` kotlin
val items = listOf("apple","banana","kiwi")
for(item in items){
  println(item)
}
```



### 使用 while 循环

``` kotlin
val items = listOf("apple","banana","kiwi")
var index = 0
while(index < item.size){
  println("item at $index is ${items[index]}")
  index++
}
```



## 使用 when 表达式

``` kotlin
fun describe(obj: Any): String = 
when(obj){
  1			-> "One"
  "Hello"	-> "Greeting"
  is Long	-> "Long"
  !is String-> 'Not a string'
  else		-> "Unknown"
}
```



### 使用范围

在范围内

``` kotlin
val x = 10
val y = 9
if(x in 1..y+1){
  println("fits in range")
}
```

不在范围内

``` kotlin
val list = listOf("a","b","c")
if(-1 !in 0..list.lastIndex){
  println("-1 is out of range")
}
if(list.size !in list.indices){
  println("list size is out of valid indices range too")
}
```

在迭代器中使用范围

``` kotlin
for(x in 1..10 step 2){
  print(x)
}
println()
for(x in 9 downTo 0 step 3){
  print(x)
}
```



### 使用集合

``` kotlin
for(item in items){
  println(item)
}
```

检查集合是否包含指定元素：

``` kotlin
when {
  "orange"	in items -> println("juicy")
  "apple"	in items -> println("apple is fine too")
}
```

使用 lambda 表达式去操作集合：

``` kotlin
fruits
.filter {it.startsWith("a")}
.sortedBy {it}
.map {it.toUpperCase()}
.forEach{println(it)}
```



### 创建类实例

``` kotlin
val rectangle = Rectangle(5.0,2.0)
val triangtle = Triangle(3.0,5.0)
//不需要使用 new
```

