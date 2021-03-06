# 惯用语法



### 创建 DTOs（POJOs/POCOs）

``` kotlin
data class Customer(val name: String,val email: String)
```

为 `Customer` 类提供以下方法：

* 为所有成员变量提供 getters（当为 var 时会提供 setters）
* `equals()`
* `hashCode()`
* `toString()`
* `copy()`
* 所有成员变量 `component1()`，`component2()` 等等 



### 方法参数默认值

``` kotlin
fun foo(a: Int = 0,b: String = ""){}
```



### 列表过滤

``` kotlin
val positives = list.filter {x -> x > 0}
```



### 字符串插值

``` kotlin
println("Name $name")
```



### 实例检查

``` kotlin
when(x){
  is Foo -> "foo"
  is Bar -> "bar"
  else	 -> "other"
}
```



### 遍历 map/list

``` kotlin
for((k,v) in map){
  println("$k -> $v")
}
```



### 使用范围

``` kotlin
for(i in 1..100 step 10){
  println("$i")
}
```



### 只读列表

``` kotlin
val list = listOf("a","b","c")
```



### 只读 map

``` kotlin
val map = mapOf("a" to 1,"b" to 2,"c" to 3)
```



### 访问 map

``` kotlin
println(map["key"])
map["key"] = value
```



### 懒加载的成员变量

``` kotlin
val p : String by lazy{
  //返回值
  "Load"
}
```



### 扩展方法

``` kotlin
fun String.spaceToCamelCase() {}
"Convert this to camelcase".spaceToCamelCase()
```



### 创建单例

``` kotlin
object Resource{
  val name = "Name"
}
```



### 如果不等于空的快捷方式

``` kotlin
val files = File("Text").listFiles()
println(files?.size)
```



### 如果不等于空否则的快捷方式

``` kotlin
val files = File("Text").listFiles()
println(files?.size ?: "empty")
```



### 如果等于空，则执行

``` kotlin
val values = ""
val email = values["email"] ?: throw IllegalStateException("Email is missing!")
```



### 如果不等于空，则执行

``` kotlin
val value = ""
value?.let{
  //
}
```



### 可空的值如果不等于空

``` kotlin
val value = ""
val mapped = value?.let{ transformValue(it)} ?: defaultValueIfValueIsNull
```



### 使用 when 返回值

``` kotlin
fun transform(color: String): Int{
  return when(color){
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
  }
}
```



### try/catch 表达式

``` kotlin
fun test(){
  val result = try{
    count()
  }catch(e: ArithmeticException)}
	throw IllegalStateException(e)
}
```



### if 表达式

``` kotlin
fun foo(param: Int){
  val result = if(param == 1){
    "one"
  }else if(param == 2){
    "two"
  }else {
    "three"
  }
}
```



### 在返回 Unit 的方法上使用 Builder 

``` kotlin
fun arrayOfMinusOnes(size: Int): IntArray{
  return IntArray(size).apply { fill(-1) }
}
```



### 单个表达式方法

``` kotlin
fun theAnswer() = 42
```

这相当于

``` kotlin
fun theAnswer(): Int{
  return 42
}
```

和 when 配合使用

``` kotlin
fun transform(color: String): Int = when(color){
  "Red"		-> 0
  "Green"	-> 1
  "Blue"	-> 2
  else		-> throw IllegalArgumentException("Invalid color param value")
}
```



### 使用 with 在单个实例上调用多个方法

``` kotlin
class Turtle {
  fun penDown()
  fun penUp()
  fun turn(degree: Double)
  fun forward(pixels: Double)
  
  val myTurtle = Turtle()
  with(myTurtle){
    penDown()
    for(i in 1..4){
      forward(100.0)
      turn(90.0)
    }
    penUp()
  }
}
```



### Java7 访问资源

``` kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use{ reader ->
                               println(reader.readText())
}
```



### 泛型方法

``` kotlin
inline fun<reified T: Any> Gson.fromJson(json: JsonElement): T = 
this.fromJson(json,T::class.java)
```



### 使用可空的 Boolean

``` kotlin
val b: Boolean? = ...
if(b == true){
  
}else{
  // b is false or null
}
```



