开发人员编写的是Java源代码文件(**.java**)，Java编译器把Java源代码编译成平台无关的字节码(**byte code**)，以类文件(**.class**)的形式保存在磁盘上。Java虚拟机(**JVM**)会负责把Java字节码加载并执行。

Java类文件中包含的字节代码可以被不同平台上的JVM所使用。Java字节代码不仅可以以文件形式存在于磁盘上，也可以通过网络方式来下载，还可以只存在于内存中。JVM中的类加载器(**ClassLoader**)会负责从包含字节码的字节数组(**byte[]**)中定义出Java类。在某些情下，可能需要动态的生成Java字节代码，或是对已有的Java字节代码进行修改。这个时候就需要用到**动态编译Java源文件**。

## 动态编译Java源文件

1. 可以使用JDK6的JavaCompiler类，来动态编译Java源代码，来生成class类文件。
2. 可以使用JDK工具类`com.tools.javac.Main`，不过该工具类只能编译存放在磁盘上的文件
3. 另外一个可用的工具是[Eclipse JDK Core](http://www.eclipse.org/jdt/core/)提供的编译器。

**使用这些动态编译的方式的时候，需要确保JDK的tools.jar在应用的CLASSPATH中**

```java
//获取系统Java编译器
        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        //获取标准Java文件管理器
        StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null);
        //设置编译成功的class文件输出路径(这里为项目根目录下的"target/classes")
        fileManager.setLocation(StandardLocation.CLASS_OUTPUT, Arrays.asList(new File("target/classes")));
        StringSourceJavaObject sourceJavaObject = new CompilerTest.StringSourceJavaObject("Main", JAVA_SOURCE);
        Iterable fileObjects = Arrays.asList(sourceJavaObject);
        //编译任务
        JavaCompiler.CompilationTask task = compiler.getTask(null, fileManager, null, null, null, fileObjects);
        //编译结果
        boolean result = task.call();
```

## Java字节码增强

Java字节码增强指的是在Java字节码生成之后，对其进行修改，增强其功能。Java字节码增强功能通常与Java源文件中的注解**Annotation**一块使用。注解在Java源码中声明了需要增强的行为及相关的元数据，由框架在**运行时刻完成对字节码代码的增强**。

一个Java类或接口的字节代码的组织形式。

```java
类文件 {
   0xCAFEBABE，小版本号，大版本号，常量池大小，常量池数组，
   访问控制标记，当前类信息，父类信息，实现的接口个数，实现的接口信息数组，域个数，
   域信息数组，方法个数，方法信息数组，属性个数，属性信息数组
}
```



如上所示，一个类或接口的字节代码使用一种松散的组织结构，其中所包含的内容依次排列。对于可能包含多个条目的内容，是以数组来表示。而在数组之前的是该数组中条目的个数。

涉及对字节代码进行操作的库有：

1. [ASM](http://asm.ow2.org/)
2. [cglib](https://github.com/cglib/cglib)
3. [Javassist](http://jboss-javassist.github.io/javassist/)
4. [BCEL](http://commons.apache.org/proper/commons-bcel/)

对类文件进行增强的时机是需要在Java源代码编译之后执行，在JVM执行之前。

1. 由IDE在完成编译操作之后执行。
2. 在构建过程中完成。
3. 实现自定义的ClassLoader。当前获取Java类的字节码之后，先进行增强处理，再从修改过的字节代码中定义出Java类。
4. 通过Java SE5引入的`java.lang.instrument`包来完成。

## Java Instrumentation

Java Instrumentation指的是可以用独立于应用程序之外的代理(**agent**)程序来检测和协助运行在JVM上的应用程序。这种检测和协助包括但不限于获取JVM运行时状态，替换和修改类定义等。

Java SE6中存在两种应用Instrumentation的方式。`permain`(**命令行**)和`agentmain`(**运行时**)。

### premain方式

在Java SE5时代，Instrument只提供了`premain`一种方式，即在真正的应用程序(包含`main`方法的程序)`main`方法启动前启动一个代理程序。

```
java -javaagent:agent_jar_path[=options] java_app_name
```

可以启动名为`java_app_name`的应用之前启动一个`agent_jar_path`指定位置的agent jar。要实现这样的一个agent.jar包，必须满足两个条件:

1. 在这个jar包的manifes文件中包含`Premain-Class`属性，并且值为代理类的全路径

2. 代理类必须提供一个

   ```Java
   public static premain(String args,Instrumentaion inst)
   ```

   或

   ```Java
   public static void premain(String args)
   ```

当在命令行启动该代理jar时，VM会根据manifest中指定的代理类，使用于main类相同的系统类加载器（即ClassLoader.getSystemClassLoader()获得的加载器）加载代理类。在执行main方法前执行premain()方法。如果premain(String args, Instrumentation inst)和premain(String args)同时存在时，优先使用前者。其中方法参数args即命令中的options，类型为String（注意不是String[]），因此如果需要多个参数，需要在方法中自己处理（比如用";"分割多个参数之类）；inst是运行时由VM自动传入的Instrumentation实例，可以用于获取VM信息。

```java
public class Agent {

    public static void premain(String args, Instrumentation instrumentation){
        System.out.println("I am agent!!!");
        //添加转化类
        instrumentation.addTransformer(new TestTransformer());
    }
}
```

### agentmain方式

Java SE6开始，提供了在应用程序的VM启动后再动态添加代理的方式。与**premain**方式类似，同样需要提供一个agent.jar，这个jar需要满足:

1. 在manifest中指定`Agent-Class`属性，值为代理类的全路径
2. 代理类需要提供

```java
public static void agentmain(String args,Instrumentation inst)
```

```java
public static void agentmain(String args)
```

如果两者都定义了，以前者优先。

Attach API中的VirtualMachine代表一个运行中的VM。其提供了loadAgent()方法，可以在运行时动态加载一个代理jar。

```java
public class Test {
    public static void main(String[] args) throws IOException, AgentLoadException, AgentInitializationException, AttachNotSupportedException {
        //连接到目标Java虚拟机
        VirtualMachine vm = VirtualMachine.attach(args[0]);
        //动态加载代理
        vm.loadAgent("/Users/linxiaotao/Documents/work/Personal/java_showcase/loadagent.jar");
    }
}
public class LoadedAgent {

    //会自动调用的代理方法
    public static void agentmain(String args, Instrumentation instrumentation){
        //打印类名
        Class[] classes = instrumentation.getAllLoadedClasses();
        for (Class clazz : classes)
            System.out.println(clazz.getName());
    }
}
```

