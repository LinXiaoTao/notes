#### 简介

Java SE5 发行版中增加了**泛型(Generic)**。

声明中具有一个或者多个**类型参数(type parameter)**的类或者接口，就是泛型类或者接口。

每种泛型定义一组**参数化的类型(parameterized type)**。

每个泛型都定义一个**原生态类型(raw type)**，即不带任何实际类型参数的泛型名称。例如:List<String>对应的原生态类型是List。

泛型方法的定义:

```java
private <U> void method(U u);
```



#### Type Parameter 和 Type Argument

``` java
//泛型类
class Box<T>{
  
}
//T在Box<T>中称为Type Parameter
//String在Box<String>中称为Type argument
Box<String> stringBox = new Box<>();

```

#### Parameterized Types

将`List<String>`称为参数化类型(Parameterized Types)。



#### 有届类型参数(Bounded Type Parameters)

``` java
//泛型类
public class Box<T extends Number>{
  
}

//泛型方法
public <U extends Number> void inspect(U u);
```

类型参数可以有**多重边界(multiple Bounds)**。

``` java
<T extends B1 & B2 & B3>
//需要注意的是，如果其中有一个边界是一个类，则必须首先指定
class B1
interface B2
interface B3
//如果写成这样，则编译报错
<T extends B2 & B1 & B3>
```



#### 通配符

1. 上边界通配符(Upper Bounded Wildcards)

   ``` java
   public static void process(List<? extends Foo> list);
   ```

2. 无界通配符(Unbounded Wildcards)

   无界通配符类型是使用通配符(?)指定的，例如`List<?>`。使用无界通配符，表示使用者并不关心具体的类型。在下面两种情况下，使用它是有意义的:

   1. 如果你正在编写可以使用`Object`类中提供的方法
   2. 当代码使用泛型类中不依赖于类型参数的方法。例如:`List.clear()`等

   考虑下面的代码:

   ``` java
   public static void printList(List<Object> list){
     
   }
   //下面这两种使用是编译不通过的，因为泛型不是协变的。
   //printList(List<Integet>);fail
   //printList(List<String>);fail
   ```

   但如果我们将参数改为`List<?>`，考虑下面的代码:

   ``` java
   public static void printList(List<?> list){
     
   }

   printList<List<Integer>);//true
   printList<List<String>);//true
   ```

   通配符一定程度上能解决**泛型不是协变**这个特性带来的一些不方便。

   需要注意的是，`List<Object>`和`List<?>`是不一样的，你可以将`Object`或者它的任意子类插入到`List<Object>`，但不能将除了`null`以外的类插入到`List<?>`。

3. 下边界通配符(Lower Bounded Wildcards)

   ``` java
   public static void process(List<? super Foo> list);
   ```

4. 通配符和子类型

   在泛型中，即使类型参数为子类关系，参数化类型也不为子类关系。即

   ``` java
   List<Double> doubleList = new Array<>();
   List<Number> numberList = new Array<>();
   //numberList = doubleList; fail
   ```

   在这里，通配符是个例外。考虑下面的代码:

   ``` java
   List<Double> doubleList = new Array<>();
   List<Number> numberList = new Array<>();
   numberList = doubleList;//true
   ```

   ![img](https://docs.oracle.com/javase/tutorial/figures/java/generics-listParent.gif)

   ​

![img](https://docs.oracle.com/javase/tutorial/figures/java/generics-wildcardSubtyping.gif)

5. 通配符捕获(Wildcard Capture)

   在某些情况下，编译器会推断通配符的类型。例如，列表可以被定义为List<?>，编译器从代码中推断出特定类型，这种情况称为**通配符捕获(Wildcard Capture)**。

   考虑下面的代码:

   ``` java
   //下面的代码是编译不通过的
   //void foo(List<?> list){
   //		list.set(0,list.get(0));  
   //}

   //这样是可以通过编译的
   private foo(List<?> list){
     fooHelp(list);
   }

   //俘获助手
   private <T> void fooHelp(List<T> list){
     list.set(0,list.get(0));
   }
   ```

   正如上面所展示的，编译器有时候并不能正确推断通配符类型，会产生**捕获错误(capture error)**。

   我个人认为，之所以会出现捕获错误，应该跟通配符是**协变**有关。考虑下面的代码:

   ``` java
   List<? extends Integer> integers = new ArrayList<>();
   List<? extends Double> doubles = new ArrayList();
   List<? extends Number> numbers = doubles;//ture
   numbers = integers;//true
   //integers.add(new Integer(10)); fail
   //integers.add(new Object()); fail
   integers.add(null);//true
   Integer integer = integers.get(0);//ture
   ```

   使用通配符的参数化类型不能加入除了`null`之外的任何值，可以理解为**一定程度上的只读**，包括`Object`。但可以将值正确取出来。

6. 通配符使用指南

   **In**变量提供数据

   **Out**变量保存数据

   1. 使用`extends`关键字，使用上边界通配符定义**In**变量
   2. 使用`super`关键字，使用下边界通配符定义**Out**变量
   3. **In**变量访问`Obejct`类方法，使用无界通配符
   4. 变量既作为**In**，又作为**Out**，不使用通配符。

   ​

#### 类型推断(Type Inference)

1. 泛型方法

   默认情况下，Java编译器会帮我们做**类型推断(Type Inference)**，这里是根据参数来推断，例如:

   ``` java
   method("Hello Java");
   //U 推断为 String
   ```

   我们也依然可以显示的指定类型参数，例如:

   ``` java
   //非静态(static)方法
   this.<String>method("Hello Java");
   //静态方法
   ClassName.<String>method("Hello Java");
   ```

2. 泛型类构造函数(Java SE 7)

   泛型类也可以使用**类型推断**，只要编译器可以从上下文推断类型参数，就可以使用空的一组类型参数(**<>**)替换调用泛型构造函数所需的类型参数(Type Variable)。这对尖括号称为**the diamond**。

   ``` java
   //不使用类型推断的构造函数
   Map<String,List<String>> map = new HashMap<String,List<String>>();
   //使用类型推断
   Map<String,List<String>> map = new HashMap<>();
   //使用类型推断必须使用Diamond，不然会有一条unchecked conversion warning
   //因为这样使用的是HashMap的原始类型，而非泛型
   Map<String,List<String>> map = new HashMap();
   ```

3. 目标类型(Target Types Java SE7)

   Java编译器利用目标类型来推断泛型方法调用的类型参数。

   ``` java
   static <T> List<T> emptyList();
   //推断
   List<String> listOne = Collections.emptyList();
   //JavaSe7之前的版本，或者显示指定类型
   List<String> listOne = Collections.<String>emptyList();
   ```

   请考虑以下方法:

   ``` java
   void processStringList(List<String> stringList){
     //process
   }
   //在Java SE7中这样调用会报错 "List<Object> cannot be converted to List<String>"
   processStringList<Collections.emptyList());
   //必须指定显示指定类型参数的值
   processStringList<Collections.<String>emptyList());
   ```

   但在Java SE8中不再需要这样做了，目标类型的概念已经被扩展为包括方法参数，在Java SE8中这样的写法是正确的:

   ``` java
   //Java SE8能正确推断方法参数的目标类型
   processStringList<Collections.emptyList());
   ```



#### 泛型不是协变的。

考虑下面的代码:

``` java
public void boxTest(Box<Number> n){
  
}

//Java SE7 构造函数类型推断，可以使用<>
Box<Integer> integerBox = new Box<>();
Box<Double> doubleBox = new Box<>();
Box<Number> numberBox = new Box<>();

//因为Integer和Double是Number的子类型，所以下面使用是正确的
numberBox.add(10);//ok
numberBox.add(10.0);//ok

//然而下面这两种写法却是编译不通过的，因为泛型不是协变的，integerBox和doubleBox和numberBox属于三种不同类型，即使Integer和Double是Number的子类型
// boxTest(integerBox);fail
// boxTest(doubleBox);fail

```

![img](https://docs.oracle.com/javase/tutorial/figures/java/generics-subtypeRelationship.gif)

#### 泛型的子类型(Subtyping)

可以通过扩展(extends)或实现(implements)泛型类或泛型接口来作为泛型的子类型。

比如对于ArrayList<E>，它继承了List<E>，而List<E>继承了Collection<E>，所以ArrayList<String>是List<String>的子类型，也是Collection<String>的子类型，只要不改变类型参数的值(String)。

![img](https://docs.oracle.com/javase/tutorial/figures/java/generics-sampleHierarchy.gif)

#### 类型擦除

Java中的泛型信息基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会被编译器在编译的时候去掉。这个过程称为**类型擦除**。

考虑下面的代码:

``` java
public class GenericTest<T> {
    private T mValue;

    public GenericTest(T value) {
        mValue = value;
    }

    public T getValue() {
        return mValue;
    }

    public void setValue(T value) {
        mValue = value;
    }
}

//简化了生成的字节码
public class com/leo/generics/GenericTest {
  //...
  private Ljava/lang/Object; mValue
  //...
  public getValue()Ljava/lang/Object;
  //...
  public setValue(Ljava/lang/Object;)V
  //....
}
```

可以看出在生成的字节码中，类型参数已经被擦除为`Object`。

#### 桥接方法(Bridge Methods)

考虑下面代码:

``` java
MyNode mn = new MyNode(5);
Node n = mn;
//当类型擦除后，Node方法为setData(Object)，MyNode方法为setData(Integer)，方法签名是不匹配的，之所以，这里调用setData(Object)能作用到MyNode的setData(Integer)，是因为编译器自动帮我们生成一个桥接方法
n.setData("Hello");
Integer x = mn.data;


//简化字节码 MyNode.class，可以看出会生成两个方法，其中setData(Object)会将调用桥接到setData(Integer)中
public void setData(java.lang.Integer);
// ...
  public void setData(java.lang.Object);
         5: invokevirtual #6                  // Method setData:(Ljava/lang/Integer;)V
         8: return

```

 #### 不可具体化类型

所谓的可具体化类型，就是在整个运行时都可以知道其类型信息的类型，包括基本类型，非泛型类型，原始类型和调用的非有届通配符。

不可具体化类型是指其类型信息在编译时被类型擦除机制擦除。我们无法在运行时知道一个不可具体化类型的所有信息。例如:`List<String>`和`List<Number>`，JVM在运行时无法分辨两者。

#### 堆污染(Heap Pollution)

当一个参数化类型变量引用了一个对象，而这个对象并非此变量的参数化类型时，堆污染就会发生。

#### 具有不可具体化的形参在可变参数的方法中的潜在隐患

具有可变参数的泛型方法可能引起堆污染。

考虑下面的代码:

``` java
public static void faultyMethod(List<String>... l) {
    Object[] objectArray = l;     // Valid
    objectArray[0] = Arrays.asList(42);
    String s = l[0].get(0);       // ClassCastException thrown here
  }
```

编译器会将可变参数方法，转换为一个数组。但我们知道的，**Java不能生成(new)参数化类型数组**。所以这里可以将`l`赋值给`Object[]`，可以会引起潜在的堆污染。

使用来自不可具体化形参的可变参数方法，会有一个堆污染的警告，如果你确保这些操作是安全的，可以使用`@SafeVarargs`屏蔽这些警告。



