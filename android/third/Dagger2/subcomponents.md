# Subcomponents

Subcomponents 是继承和扩展 parent component 的 object graph。你可以使用它们将应用程序的 object graph 划分为 subgraphs，以将应用程序的不同部分彼此封装起来，或者在 component 中使用多个 scope。

绑定在 subcomponent 中的对象可以依赖于绑定在 parent component 中的任何对象，或者任何 ancestor component，添加在 subcomponent 自身绑定 modules 的对象。