---
title: 关于Java字节码学习的小例子
tags: java
date: 2017-04-29 15:55:27
---


首先，我们来看下这段代码：

``` java
private static int test() {
        int i = 0;
        try {
            return i;
        } finally {
            ++i;
        }
    }
```

在不运行代码的情况，你会觉得方法应该返回什么？可能会有很多高手一眼就能看出，但对于博主这样的菜鸡来说，大概猜是返回 0 ，但又不敢肯定。这个时候，我们就可以通过分析它生成的字节码文件来看看 JVM 是如何处理这段代码的。

使用：

`javac Test.java`

`javap -v -p Test`

> `-p` 是将私有方法等等也打印出来

通过上面两个命令，我们可以看到字节码的真面目了，省去无关紧要的片段，核心片段如下：

```
		 0: iconst_0
         1: istore_0
         2: iload_0
         3: istore_1
         4: iinc          0, 1
         7: iload_1
         8: ireturn
         9: astore_2
        10: iinc          0, 1
        13: aload_2
        14: athrow
```

没接触过字节码的同学可能会有点傻眼，博主第一次看的时候也觉得像天书一般，下面我们来对每条指令做一一讲解：

```
		 // 0 - 8 是正常流程
         0: iconst_0  //将 0 加载到堆栈上
         1: istore_0  //将 int 值存储到变量 0 变量 0 = 0
         2: iload_0   //从局部变量 0 加载一个 int 值
         3: istore_1  //将 int 值存储到变量 1 变量 1 = 0
         4: iinc          0, 1  //将变量 0 中保留的 int 值增加 1 变量 0 = 1
         7: iload_1   //从局部变量 1 加载一个 int 值
         8: ireturn   //返回一个 int 值 实际这里返回局部变量 1 值为 0

         9: astore_2  //将引用存储到变量 2 变量 2 = 1
        10: iinc          0, 1  //变量 0 增加 1 变量 0 = 1
        13: aload_2   //从变量 2 加载到堆栈上
        14: athrow    //抛出错误或异常  
```

通过上面对每条命令的注释，我们可以得到结论：**方法返回局部变量 1 的值**，而这个值的变量为 0。

对上面理解的同学可以继续往下看，以免看了下面这个例子后更混淆。

——————————————————————————分隔符————————————————————————

``` java
private static Person test(Person person) {
        person.age = 0;
        try {
            return person;
        } finally {
            person.age++;
        }
    }
private static class Person {
        int age;
    }
```



看下上面的例子，你觉得会输出什么呢？这次我们也是直接看字节码。

```
 		 // 0 - 18 为正常流程
 		 0: aload_0		//将变量 0 加载到堆栈上
         1: iconst_0	//加载 int 值 0 到堆栈上
         2: putfield      #6                  // Field Test$Person.age:I	//设置对象的字段值
         5: aload_0		//将变量 0 加载到堆栈上 
         6: astore_1	//将变量 0 储存到变量 1	
         7: aload_0		//将变量 0 加载到堆栈上 
         8: dup			//复制堆栈顶部的值
         9: getfield      #6                  // Field Test$Person.age:I	//获取对象的字段值
        12: iconst_1	//加载 int 值 1 到堆栈上
        13: iadd		//两个 int 值相加
        14: putfield      #6                  // Field Test$Person.age:I	//设置对象的字段值 
        17: aload_1		//将变量 1 加载到堆栈上
        18: areturn		//返回堆栈顶部 实际这里返回的变量 1 
        
        19: astore_2
        20: aload_0
        21: dup
        22: getfield      #6                  // Field Test$Person.age:I
        25: iconst_1
        26: iadd
        27: putfield      #6                  // Field Test$Person.age:I
        30: aload_2
        31: athrow
```

分别运行上面两段代码片段，会发现两种不同的结果，第一个代码片段会输出 **0** ，而第二个则会输出 **1**，通过对上面字节码的分析，可以看到一些共同点，即两个方法都是返回预先拷贝的副本，区别在于，第二个代码片段中，拷贝的是类对象，而在 Java 中，是通过引用是持有对象的，两个不同的引用可以指向同一个对象，所以对**变量 0 **的操作也会影响到**变量1**，而在第一个代码片段中，`int` 是基本类型，并不是类。

> 注：这里对象指通过 `new class` 去生成类的实例。

这里有个要注意的地方，像 `Integer`，`String` 这些都是不可变的对象，即通过构造方法生成对象后，任何的改变实际上是又生成了一个新的对象，比如：

``` java
 private static String test(String string) {
        string = "Change";
        return string;
    }
private static Integer test(Integer integer) {
        integer++;
        return integer;
    }
```

所以当你写了这样的代码：

``` java
private static Person test(Integer integer) {
        try {
            return integer;
        } finally {
            integer++;
        }
    }
```

根据上面的分析，返回的是对象拷贝，你可能会认为值会被改变，实际上，`integer++` 操作的是新的对象，并不会影响参数对象。