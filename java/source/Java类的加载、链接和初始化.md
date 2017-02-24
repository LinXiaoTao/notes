# Java类的加载、链接和初始化

---

[参考地址](http://www.infoq.com/cn/articles/cf-Java-class-loader#)
[参考地址](https://github.com/JustinSDK/JavaSE6Tutorial/blob/master/docs/CH16.md)
[参考地址](https://www.ibm.com/developerworks/cn/java/j-lo-classloader/)
[参考地址](http://blog.csdn.net/xyang81/article/details/7292380)

Java类从字节码到能够在JVM中使用,需要经过**加载**,**链接**和**初始化**这三个步骤.
这三个步骤对开发人员直接可见的是Java类的**加载**,通过使用Java类加载器(`ClassLoader`)可以在运行时动态的加载Java类,而**链接**和**初始化**则是在使用Java类之前会发生的动作.

* Java类的加载
 * Java类加载是由类加载器来完成的.一般来说,类加载器分为两类:
  1. 启动类加载器(bootstrap)
  2. 用户自定义的类加载器(user-definded)
 
 两者的区别在于启动类加载器是由JVM的原生代码实现的,而用户自定义的类加载器都继承自Java中的`ClassLoader`类.
 * Java默认提供的三个ClassLoader
  1. BootStrap ClassLoader:称为**启动类加载器**,是Java类加载层次中最顶层的类加载器,**负责加载JDK中的核心库**
  2. Extended ClassLoader:称为**扩展类加载器**,负责加载Java的扩展类库,默认加载JAVA_HOME/jre/lib/ext目录下的所有    jar
  3. App ClassLoader:称为**系统类加载器**,负责加载应用程序classpath目录下的所有jar和class文件
 * 默认ClassLoader载入流程
  启动JVM进行初始化,接着产生BootStrap Loader,BootStrap Loader会载入Extened Loader,并设定Extened    Loader的parent为BootStrap Loader,接着BootStrap Loader会载入System Loader,并将System  Loader的parent设定为Extended Loader.BootStrap Loader通常由C编写,Extended Loader是由Java编写,实际对应于`sun.misc.Launcher\$ExtClassLoader`.System Loader是由Java编写,实际对应于`sun.misc.Launcher\$AppClassLoader`.
 
 Java类加载器有两个比较重要的特征:**层次组织结构**和**代理模式**.层次组织结构指的是每个类加载器都有一个父类加载器,通过`getParent()`方法可以获取到.**BootStrap Loader因为是由C编写,所以在Java程序中获取出来为null**.类加载器通过这种父亲-后代的方式组织在一起,形成了树状层次结果.代理模式指的是一个类加载器既可以自己完成Java类的定义工作,也可以代理给其他类加载器来完成.由于代理模式的存在，启动一个类的加载过程的类加载器和最终定义这个类的类加载器可能并不是一个。前者称为初始类加载器，而后者称为定义类加载器。两者的关联在于：一个Java类的定义类加载器是该类所导入的其它Java类的初始类加载器。比如类A通过import导入了类 B，那么由类A的定义类加载器负责启动类B的加载过程。
一般的类加载器在尝试自己去加载某个Java类之前，会首先代理给其父类加载器。当父类加载器找不到的时候，才会尝试自己加载。
类加载器的一个重要用途是在JVM中为相同名称的Java类创建隔离空间。在JVM中，判断两个类是否相同，不仅是根据该类的二进制名称，还需要根据两个类的定义类加载器。只有两者完全一样，才认为两个类的是相同的。
      
* Java类的链接
Java类的链接指的是将Java类的二进制代码合并到JVM的运行状态之中的过程.在链接之前,类必须被成功加载.
类的链接包括**验证**,**准备**和**解析**等几个过程.验证是用来确保Java类的二进制表示在结构上是完全正确的.准备过程则是创建Java类中的静态域,并将这些域设置为默认值.**准备过程中不会执行代码**.解析过程是确保类中包含的对其他类或接口的形式引用,包含它的父类,所实现的接口,方法的形式参数和返回值的Java类等,被正确的找到.**解析过程可能会导致其他Java类被加载**.
不同的JVM可能会选择不同的解析策略.一种做法是在链接的时候,就递归的把所有依赖的形式引用都进行解析.而另外的做法则可能是只有在真正需要的时候才进行解析.

* Java类的初始化
当一个Java类第一次被真正使用的时候,JVM会进行该类的初始化操作.初始化过程的主要操作是**执行静态代码块和初始化静态域**.**在一个类初始化之前,它的直接父类也需要被初始化.而接口不会.初始化时候,会按照源码从上到下依次执行初始化操作**
Java类和接口的初始化的时机包括:
 * 创建Java类实例:
 ``` java
 MyClass obj = new MyClass();
 ```
 * 调用类中的静态方法:
 ``` java
 MyClass.sayHello();
 ```
 * 给Java类或接口中声明的静态域赋值:
 ``` java
 MyClaa.valu = 10;
 ```
 * 访问Java类或接口中声明的静态域,并且**该域不是常值变量**:
 ``` java
 class Myclass{
    static{
        System.out.println("静态代码块初始化");
    }
    public static final VALUE = "Hello";
    public static sValue = "Hello";
    public static final HELLO = getValue();
    
    public static String getValue(){
        return "Hello";
    }
    
    public static void main(String[] args){
        int value = MyClass.VALUE;//不会输出"静态代码块初始化"
        value = MyClass.sValue;//会输出"静态代码块初始化"
        value = MyClass.HELLO;//会输出"静态代码块初始化"
    }
 }
 ```
 * 使用反射API也可能造成类或接口的初始化
 ``` java
 //会导致初始化
 @Test
    public void testReflectionInit(){
        try {
            Class clazz = Class.forName("com.leo.initialization.Two");
            Assert.assertNotNull(clazz);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
 
 //不会导致初始化
 @Test
    public void testReflectionNotInit(){
        try {
            Class clazz = Class.forName("com.leo.initialization.Two",false,this.getClass().getClassLoader());
            Assert.assertNotNull(clazz);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
 ```
     **需要注意的是,当访问一个Java类或接口中的静态域的时候,只有真正声明这个域的类或接口才会被初始化**
     ``` java
     public class One {
        static int value = 100;
    
        static {
            System.out.println("One静态代码块");
        }
    
        public One() {
            System.out.println("One构造方法");
        }
    }
    public class Two extends One {
    
        public static String sValue;
        public static final String VALUE = "Hello";
    
        static {
            System.out.println("Two静态代码块");
        }
    
        public Two() {
            System.out.println("Two构造方法");
        }
    }
    @Test
        public void testOneIsInit(){
            Assert.assertEquals(Two.value,100);//输出"One静态代码块"
        }
     ```
     **当某个Java类初始化,会先初始化它的父类,如果有的话**
     ``` java
     public class One{
        static{
            System.out.println("One的静态代码块");
            }
     }
     public class Two extends One{
        static{
            System.out.println("Two的静态代码块");
            }
     }
     public class Test{
        public static void main(String[] args){
                Two two = new Two();//会先输出"One的静态代码块",再输出"Two的静态代码块"
            }
     }
     ```
     **接口的初始化,则不会初始化它的父类,如果有的话**
     ``` java
    interface I {
    int i = 1, ii = Test.out("ii", 2);
         }
    interface J extends I {
        int j = Test.out("j", 3), jj = Test.out("jj", 4);
         }
    interface K extends J {
        int k = Test.out("k", 5);
            }
    class Test {
        public static void main(String[] args) {
            System.out.println(J.i);
            System.out.println(K.j);
            //会输出
            //1
            //j=3
            //jj=4 这里会输出,是因为调用Test.out("j",3),该值不是常数,会导致初始化
            //3
            }
        static int out(String s, int i) {
            System.out.println(s + "=" + i);
            return i;
            }
    }
     ```
     **需要特别注意的是,如果接口中存在Java8新增的`default method`的话,即使访问的是,接口的常数变量,也会导致初始化**
     ``` java
     //将上个例子中的I修改
     public interface I {
     int i = 1, ii = Test.out("ii", 2);

     default void method() {}
     }
     //再次调用
     System.out.println(J.i);
     System.out.println(K.j);
     //输出
     1
    ii=2//I接口会被初始化
    j=3
    jj=4
    3
     ```



