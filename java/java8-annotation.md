[j-lo-java8annotation](https://www.ibm.com/developerworks/cn/java/j-lo-java8annotation/index.html)

[ma14-architect-annotations-2177655](http://www.oracle.com/technetwork/articles/java/ma14-architect-annotations-2177655.html)

Java 在 JDK 1.5 中开始引入 Annotation 功能，

> 注解对代码操作没有直接的影响，但在处理时，它们可以为注解处理器提供必要的信息。

Java 8 中对 Annotation 引入了两项更新：Type Annotation 和 Repeating Annotation。



## Type Annotation

在 Java 8 之前的版本中，只能允许在声明式前使用 Annotation。而在 Java 8 版本中，Annotation 可以被用在任何使用 Type 的地方，例如：初始化对象时 (new)，对象类型转化时，使用 implements 表达式时，或者使用 throws 表达式时。

``` java
String text = new @NotNull String();

text = (@NotNull String) str;

class MyList<T> implements @ReadOnly List<@ReadOnly T> {
  
}

public void validateValuies() throws @Critical ValidationFailedException {
  
}
```

定义一个 Type Annotation 的方法与普通的 Annotation 类似，只需要指定 Target 为 `ElementType.TYPE_PARAMETER` 或者 `ElementType.TYPE_USE`，或者同时指定这两个 Target。

``` java
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@interface TestTypeAnnotation {

}
```

`ElementType.TYPE_PARAMETER` 表示，该注解可以用于泛型的类型参数声明处：

``` java
static class TestList<@TestTypeAnnotation T> extends ArrayList<T> {
}
```

`ElementType.TYPE_USE` 表示，该注解可以用在所有使用 Type 的地方。

* 生成新的对象。类型注解可以在创建新对象时提供静态验证，以帮助强制对象构造函数的注释兼容性。

  ``` java
  User user = new @Interned User();
  ```

* 泛型和数组。泛型和数组非常适合 Type Annotation，因为它们可以帮助限制这些对象所期望的数据。

  ``` java
  @NotEmpty User[]
  ```

* 类型转换。类型转换可以使用注解以确保注解类型保留在转换中。他们也可以用作限定符来警告意外的转换。

  ``` java
  @Readonly Object x = (@Readonly Date) date;

  Object myObject = (@NotNull Object) obj;
  ```

* 继承。帮助实现正确的类继承或者实现。

  ``` java
  class MyForecast<T> implements @NonEmpty List< @ReadOnly T>
  ```

* 异常。确保符合标准的异常。

  ``` java
  catch (@Critical Exception e) {
   ... 
  }
  ```

* 接收器。可以通过在参数列表中明确列出方法来注解方法的接收器参数。

  ``` java
  class Weather {
      ...
      void tempCalc(@ReadOnly Weather this){}
      ...
  }
  ```




### 应用 Type Annotation

一般情况下，Type Annotation 被放置在所应用的类型之前，比如 `@ReadOnly int [] nums`，这会注解在 `int` 类型上。如果想要注解在数组上，应该被放置在类型的相关部分，`int @ReadOnly [] nums`。




## Repeating Annotation

之前版本的 JDK 并不允许开发者在同一个声明式前加注同样的 Annotation，（即使属性值不同）这样的代码在编译过程中会提示错误。而 Java 8 解除了这一限制，开发者可以根据各自系统中的实际需求在所有可以使用 Annotation 的地方使用 Repeating Annotation。

首先在需要重复标注特性的 Annotation 前加上 `@Repeatable` 标签。

``` java
@Target({ElementType.FIELD, ElementType.LOCAL_VARIABLE, ElementType.PARAMETER})
@Repeatable(Users.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface User {

    String username();

    int age();

    String address() default "None";
}
```

而 `@Repeatable` 中的值为：

``` java
@Target({ElementType.FIELD, ElementType.LOCAL_VARIABLE, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface Users {
	
  	//value 必须为返回可重复注解的数组
    User[] value();

}
```

``` java
@User(username = "leo", age = 24)
@User(username = "test", age = 18)
final static UserBean leo = new UserBean("leo", 24);
```

可以通过以下方法获取注解：

``` java
try {                                                                      
    Field field = JavaMain.class.getDeclaredField("leo");                  
    field.setAccessible(true);                                             
    User[] userAnnotation = field.getDeclaredAnnotationsByType(User.class);
    System.out.println(Arrays.toString(userAnnotation));                   
    Annotation[] annotations = field.getDeclaredAnnotations();             
    System.out.println(Arrays.toString(annotations));                      
} catch (NoSuchFieldException e) {                                         
    e.printStackTrace();                                                   
}                                                                          
```

