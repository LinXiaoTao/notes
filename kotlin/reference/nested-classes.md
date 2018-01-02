# 嵌套类与内部类



类可以嵌套在其他类中：

``` kotlin
class Other {
  private val bar: Int = 1
  class Nested {
    fun foo() = 2
  }
}
```



## 内部类

类可以标记为 *inner* 以便能够访问外部类的成员。内部类会带有一个对外部类的对象的引用：

``` kotlin
class Outer {
  private val bar: Int = 1
  inner class Inner {
    fun foo() = bar   
  }
}

val demo = Outer().Inner().foo() 
```



## 匿名内部类

使用[对象表达式](https://www.kotlincn.net/docs/reference/object-declarations.html#%E5%AF%B9%E8%B1%A1%E8%A1%A8%E8%BE%BE%E5%BC%8F)创建匿名内部类实例：

``` kotlin
window.addMouseListener(object: MouseAdapter()) {
  override fun mouseClicked(e: MouseEvent) {
    
  }
  
  override fun mouseEntered(e: MouseEvent) {
    
  }
}
```

如果对象是函数式 Java 接口（即具有单个抽象方法的 Java 接口）的实例， 你可以使用带接口类型前缀的lambda表达式创建它：

``` kotlin
val listener = ActionListener { println("clicked") }
```

