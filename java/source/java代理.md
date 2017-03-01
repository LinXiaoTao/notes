代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问。

* 静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件已经存在了。
* 动态代理：在程序运行时，运用反射机制动态创建而成

通过Proxy.newProxyInstance方法实现动态代理

```java
public static Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
```

loader:类加载器

interfaces:代理接口

h:代理处理类