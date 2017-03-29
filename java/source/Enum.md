Java SE5新增了枚举类型(Enum)，枚举类型的定义如下：

``` java
enum RainbowColor{
  RED, ORANGE, YELLOW, GREEN, CYAN, BLUE, PURPLE;
}
```

枚举会隐式继承`java.lang.Enum`，所以不能继承其他类，但可以实现接口。

Java要求在任何字段或方法之前首先定义常量。此外，当有字段和方法时，枚举常量的列表必须以分号结尾。

枚举类型的构造函数必须是包私有或私有访问。它会自动创建在枚举正文开头定义的常量。你不能自己调用枚举构造函数。

### Enum相关工具类

EnumSet 是一个针对枚举类型的高性能的 Set 接口实现。EnumSet 中装入的所有枚举对象都必须是同一种类型，其内部的实现：

* `Set.size() <= 64`则使用`RegularEnumSet`实现，其内部是使用**long**来维护。
* `Set.size() > 64`则使用`JumboEnumSet`实现，其内部则是使用**long[]**维护。

一般用法如下：

``` java
 EnumSet<Planet> enumSet = EnumSet.allOf(Planet.class);
        for (Planet planet : enumSet)
            System.out.println(planet);
```

EnumMap 也是一个高性能的 Map 接口实现，用来管理使用枚举类型作为 keys 的映射表，内部是通过数组方式来实现。

一般用法如下：

``` java
 EnumMap<Planet,String> enumMap = new EnumMap<>(Planet.class);
        enumMap.put(Planet.MERCURY,"123");
        enumMap.put(Planet.PLANET,"234");
        for (EnumMap.Entry<Planet,String> entry : enumMap.entrySet()){
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
```

