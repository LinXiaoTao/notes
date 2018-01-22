# Java 中调用 Kotlin

Java 可以轻松调用 Kotlin 代码。



### 属性

Kotlin 属性会编译成以下 Java 元素：

* 一个 getter 方法，名称通过加前缀 `get` 算出；
* 一个 setter 方法，名称通过加前缀 `set` 算出（只适用于 `var` 属性）；
* 一个私有字段，与属性名称相同（仅适用于具有幕后字段的属性）。

例如，`var firstName: String` 编译成以下 Java 声明：

``` java
private String firstName;

public String getFirstName(){
  return firstName;
}

public void setFirstName(String firstName) {
  this.firstName = firstName;
}
```

如果属性的名称以 `is` 开头，则使用不同的名称映射规则：getter 的名称与属性名称相同，并且 setter 的名称是通过将 `is` 替换为 `set` 获得。 例如，对于属性 `isOpen`，其 getter 会称做 `isOpen()`，而其 setter 会称做 `setOpen()`。 这一规则适用于任何类型的属性，并不仅限于 `Boolean`。



### 包级函数

在 `org.foo.bar` 包内的 `example.kt` 文件中声明的所有的函数和属性，包括扩展函数， 都编译成一个名为 `org.foo.bar.ExampleKt` 的 Java 类的静态方法。

```java
package demo

class Foo

fun bar() {
  
}
```

``` java

new demo.Foo()
demo.ExampleKt.bar()
```

可以使用 `@JvmName` 注解修改生成的 Java 类的类名：

```kotlin
@file:JvmName("DemoUtils")

package demo

class Foo

fun bar() {
  
}
```

``` java
// Java
new demo.Foo();
demo.DemoUtils.bar();
```

如果多个文件中生成了相同的 Java 类名（包名相同并且类名相同或者有相同的 `@JvmName` 注解）通常是错误的。然而，编译器能够生成一个单一的 Java 外观类，它具有指定的名称且包含来自所有文件中具有该名称的所有声明。 要启用生成这样的外观，请在所有相关文件中使用 @JvmMultifileClass 注解。

```kotlin

@file:JvmName("Utlis")
@file.JvmMultifileClass

package demo 

fun foo(){
  
}
```

``` kotlin
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun bar(){
  
}
```

``` java
// Java
demo.Utils.foo();
demo.Utils.bar();
```



### 实例字段

如果需要在 Java 中将 Kotlin 属性作为字段暴露，那就需要使用 `@JvmField` 注解对其标注。 该字段将具有与底层属性相同的可见性。如果一个属性有幕后字段（backing field）、非私有、没有 `open` /`override` 或者 `const`修饰符并且不是被委托的属性，那么你可以用 `@JvmField` 注解该属性。

```kotlin
class C(id: String) {
  @JvmField val ID = id
}
```

``` java
class JavaClient {
  public String getID(C c) {
    return c.ID:
  }
}
```

[延迟初始化的](https://www.kotlincn.net/docs/reference/properties.html#%E5%BB%B6%E8%BF%9F%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B1%9E%E6%80%A7%E4%B8%8E%E5%8F%98%E9%87%8F)属性（在Java中）也会暴露为字段。 该字段的可见性与 `lateinit` 属性的 setter 相同。



### 静态字段

在命名对象或伴生对象中声明的 Kotlin 属性会在该命名对象或包含伴生对象的类中具有静态幕后字段。

通常这些字段是私有的，但可以通过以下方式之一暴露出来：

* `@JvmField` 注解；
* `lateinit` 修饰符；
* `const` 修饰符。

使用 `@JvmField` 标注这样的属性使其成为与属性本身具有相同可见性的静态字段。

> 测试发现不起作用。

```kotlin
class key(val value: Int) {
  companion object { 
  	@JvmField
    val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value }
  }
}
```

``` java
key.COMPARATOR.comparre(key1,key2)
```

在命名对象或者伴生对象中的一个[延迟初始化的](https://www.kotlincn.net/docs/reference/properties.html#%E5%BB%B6%E8%BF%9F%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B1%9E%E6%80%A7%E4%B8%8E%E5%8F%98%E9%87%8F)属性具有与属性 setter 相同可见性的静态幕后字段。

```kotlin
object Singleton {
  lateinit var provider: Provider
}
```

``` java
Singleton.provider = new Provider();
```

用 `const` 标注的（在类中以及在顶层的）属性在 Java 中会成为静态字段：

> 伴生对象设置 const 不起作用

```kotlin
// 文件 example.kt

object Obj {
  const val CONST = 1
}

class C {
  companion object {
    const val VERSION = 9
  } 
}

const val MAX = 239
```

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
int v = c.VERSION
```



### 静态方法

如上所述，Kotlin 将包级函数表示为静态方法。 Kotlin 还可以为命名对象或伴生对象中定义的函数生成静态方法，如果你将这些函数标注为 `@JvmStatic` 的话。 如果你使用该注解，编译器既会在相应对象的类中生成静态方法，也会在对象自身中生成实例方法。 例如：

``` kotlin
class C {
  companion object {
    @JvmStatic fun foo() {}
      fun bar() {}
  }
}
```

现在，`foo()` 在 Java 中是静态的，而 `bar()` 不是：

```java
c.foo(); //ok
c.bar(); //error
C.Compainon.foo(); // ok，保留实例方法
C.Compainon.bar();//唯一的工作方式
```

对于命名对象也同样：

```kotlin
object Obj {
  @JvmStatic fun foo() {}
  fun bar() {}
}
```

``` java
Obj.foo(); // 没问题
Obj.bar(); // 错误
Obj.INSTANCE.bar(); // 没问题，通过单例实例调用
Obj.INSTANCE.foo(); // 也没问题
```

`@JvmStatic`　注解也可以应用于对象或伴生对象的属性， 使其 getter 和 setter 方法在该对象或包含该伴生对象的类中是静态成员。



### 可见性

Kotlin 的可见性以下列方式映射到 Java：

* `private` 成员编译成 `private` 成员；
* `private` 的顶层声明编译成包级局部声明；
* `protected` 保持 `protected`（注意 Java 允许访问同一个包中其他类的受保护成员， 而 Kotlin 不能，所以 Java 类会访问更广泛的代码）；
* `internal` 声明会成为 Java 中的 `public`。`internal` 类的成员会通过名字修饰，使其更难以在 Java 中意外使用到，并且根据 Kotlin 规则使其允许重载相同签名的成员而互不可见；
* `public` 保持 `public`。



### KClass

有时你需要调用有 `KClass` 类型参数的 Kotlin 方法。 因为没有从 `Class` 到 `KClass` 的自动转换，所以你必须通过调用 `Class<T>.kotlin` 扩展属性的等价形式来手动进行转换：

```kotlin
kotlin.jvm.JvmClassMappingKt.getKotlinClass(MainView.class)
```



### 用 @JvmName 解决签名冲突

有时我们想让一个 Kotlin 中的命名函数在字节码中有另外一个 JVM 名称。 最突出的例子是由于*类型擦除*引发的：

```kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

这两个函数不能同时定义，因为它们的 JVM 签名是一样的：`filterValid(Ljava/util/List;)Ljava/util/List;`。 如果我们真的希望它们在 Kotlin 中用相同名称，我们需要用 `@JvmName` 去标注其中的一个（或两个），并指定不同的名称作为参数：

``` kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValideInt")
fun List<Int>.filterValid(): List<Int>
```

在 Kotlin 中它们可以用相同的名称 `filterValid` 来访问，而在 Java 中，它们分别是 `filterValid` 和 `filterValidInt`。

同样的技巧也适用于属性 `x` 和函数 `getX()` 共存：

```kotlin
val x:Int 
	@JvmName("getX_prop")
	get() = 15

fun getX() = 10
```



### 生成重载

通常，如果你写一个有默认参数值的 Kotlin 函数，在 Java 中只会有一个所有参数都存在的完整参数签名的方法可见，如果希望向 Java 调用者暴露多个重载，可以使用 `@JvmOverloads` 注解。

该注解也适用于构造函数、静态方法等。它不能用于抽象方法，包括在接口中定义的方法。

``` kotlin
class Foo @JvmOverloads constructor(x: Int,y: Double = 0.0) {
  @JvmOverloads fun f(a: String,b: Int = 0,c: String ="abc") {
    
  }
}
```

对于每一个有默认值的参数，都会生成一个额外的重载，这个重载会把这个参数和它右边的所有参数都移除掉。在上例中，会生成以下代码 ：

```java
Foo(int x,double y){}
Foo(int x){}

void f(String a,int b,String c) {}
void f(String a,int b) {}
void f(String a) {}
```

请注意，如[次构造函数](https://www.kotlincn.net/docs/reference/classes.html#%E6%AC%A1%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)中所述，如果一个类的所有构造函数参数都有默认值，那么会为其生成一个公有的无参构造函数。这就算没有 `@JvmOverloads` 注解也有效。



### 受检异常

如上所述，Kotlin 没有受检异常。 所以，通常 Kotlin 函数的 Java 签名不会声明抛出异常。 于是如果我们有一个这样的 Kotlin 函数：

```kotlin
package demo

fun foo() {
  throw IOException()
}
```

然后我们想要在 Java 中调用它并捕捉这个异常：

```java

try {
  demo.Example.foo()
}catch(IOException e) { //error
  
}

```

因为 `foo()` 没有声明 `IOException`，我们从 Java 编译器得到了一个报错消息。 为了解决这个问题，要在 Kotlin 中使用 `@Throws` 注解。

```kotlin

@Throws(IOException::class)
fun foo() {
  throw IOException()
}

```



### 空安全

当从 Java 中调用 Kotlin 函数时，没人阻止我们将 *null* 作为非空参数传递。 这就是为什么 Kotlin 给所有期望非空参数的公有函数生成运行时检测。 这样我们就能在 Java 代码里立即得到 `NullPointerException`。



### 型变的泛型

当 Kotlin 的类使用了[声明处型变](https://www.kotlincn.net/docs/reference/generics.html#%E5%A3%B0%E6%98%8E%E5%A4%84%E5%9E%8B%E5%8F%98)，有两种选择可以从 Java 代码中看到它们的用法。让我们假设我们有以下类和两个使用它的函数：

```kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

一种看似理所当然地将这辆函数转换成 Java 代码的方式可能会是：

``` java
Box<Derived> boxDerived(Derived value) {}
Base unboxBase(Box<Base> box) {}
```

问题是，在 Kotlin 中我们可以这样写 `unboxBase(boxDerived("s"))`，但是在 Java 中是行不通的，因为在 Java 中类 `Box` 在其泛型参数 `T` 上是*不型变的*，于是 `Box<Derived>` 并不是 `Box<Base>` 的子类。 要使其在 Java 中工作，我们按以下这样定义 `unboxBase`：

```java
Base unboxBase(Box<? extends Base> box) {}
```

这里我们使用 Java 的*通配符类型*（`? extends Base`）来通过使用处型变来模拟声明处型变，因为在 Java 中只能这样。

当它*作为参数*出现时，为了让 Kotlin 的 API 在 Java 中工作，对于协变定义的 `Box` 我们生成 `Box<Super>` 作为 `Box<? extends Super>` （或者对于逆变定义的 `Foo` 生成 `Foo<? super Bar>`）。当它是一个返回值时， 我们不生成通配符，因为否则 Java 客户端将必须处理它们（并且它违反常用 Java 编码风格）。因此，我们的示例中的对应函数实际上翻译如下：

``` java
//作为返回类型-没有通配符
Box<Derived> boxDerived(Derived value) {}

//作为参数-有通配符
Base unboxBase<Box<? extends Base> box) {}
```

注意：当参数类型是 final 时，生成通配符通常没有意义，所以无论在什么地方 `Box<String>` 始终转换为 `Box<String>`。

如果我们在默认不生成通配符的地方需要通配符，我们可以使用 `@JvmWildcard` 注解：

``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
//将被转换成
//Box<? extends Derived> boxDerived(Derived value) {}
```

另一方面，如果我们根本不需要默认的通配符转换，我们可以使用`@JvmSuppressWildcards`

``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
//会被翻译为
//Base unboxBase<Box<Base> box) {}
```

注意：`@JvmSuppressWildcards` 不仅可用于单个类型参数，还可用于整个声明（如函数或类），从而抑制其中的所有通配符。



### Nothing 类型翻译

类型 [`Nothing`](https://www.kotlincn.net/docs/reference/exceptions.html#nothing-%E7%B1%BB%E5%9E%8B) 是特殊的，因为它在 Java 中没有自然的对应。确实，每个 Java 引用类型，包括`java.lang.Void` 都可以接受 `null` 值，但是 Nothing 不行。因此，这种类型不能在 Java 世界中准确表示。这就是为什么在使用 `Nothing` 参数的地方 Kotlin 生成一个原始类型：

```kotlin
fun emptyList(): List<Nothing> = listOf()
//会被翻译为
//List emptyList() {}
```

