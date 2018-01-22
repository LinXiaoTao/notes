# 类型安全的构建器



[构建器（builder）](http://www.groovy-lang.org/dsls.html#_nodebuilder)的概念在 *Groovy* 社区中非常热门。 构建器允许以半声明（semi-declarative）的方式定义数据。构建器很适合用来[生成 XML](http://www.groovy-lang.org/processing-xml.html#_creating_xml)、 [布局 UI 组件](http://www.groovy-lang.org/swing.html)、 [描述 3D 场景](http://www.artima.com/weblogs/viewpost.jsp?thread=296081)以及其他更多功能……

对于很多情况下，Kotlin 允许*检查类型*的构建器，这使得它们比 Groovy 自身的动态类型实现更具吸引力。

对于其余的情况，Kotlin 支持动态类型构建器。



### 一个类型安全的构建器示例

