# 接口

Kotlin 的接口与 Java 8 类似，既包含抽象方法的声明，也包含实现。与抽象类不同的是，接口无法保存状态。它可以有属性但必须声明为抽象或提供访问器实现。

使用关键字 interface 来定义接口：

``` kotlin
interface MyInterface {
  fun bar()
  fun foo() {
    //
  }
}
```



### 实现接口

一个类或者对象可以实现一个或多个接口。

``` kotlin
class Child : MyInterface {
  override fun bar() {
    
  }
}
```



### 接口中的属性

你可以在接口中定义属性。在接口中声明的属性要么是抽象的，要么提供访问器的实现。在接口中声明的属性不能有幕后字段，因此接口中声明的访问器不能引用它们。

``` kotlin
interface MyInterface {
  val prop: Int //抽象
  val propertyWithImplementation: String
  	get() = "Foo"
  fun foo() {
    println(prop)
  }
}

class Child : MyInterface {
  override val prop: Int = 29
}
```



### 解决覆盖冲突

实现多个接口时，可能会遇到同一方法继承多个实现的问题。例如：

``` kotlin
interface A {
  fun foo() { print("A") }
  fun bar()
}

interface B {
  fun foo() { print("B") }
  fun bar() { print("bar") }
}

class C : A {
  override fun bar() { print("bar") }
}

class D : A,B {
  override fun foo() {
    super<A>.foo()
    super<B>.foo()
  }
  
  override fun bar() {
    super<B>.bar()
  }
}
```