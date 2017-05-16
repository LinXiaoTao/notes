RecyclerView 是用于向大型数据集提供有限窗口的灵活布局。

### 术语：

* **Adapter**:  一个 `RecyclerView.Adapter` 的子类，用于管理提供代表数据集中单项数据的视图。
* **Position**: 在 **Adapter** 中单项数据的位置。
* **Index**: 在调用 `getChildAt(int)` 时使用的附加子视图的下标。与 **Position** 对比。
* **Binding**: 准备子视图以显示对应于 **Adapter** 中的 **Position** 的数据的过程。
* **Recycle(View)**: 一个之前用于特定 **Adapter Position** 的视图被放进缓存中，以便之后再次显示相同类型的数据时候使用。这可以通过跳过布局的 **inflation** 和构造来大幅提高性能。
* **Scrap(View)**: 在布局期间已进入暂时分离状态的子视图。**Scrap** 视图可能会重复使用，而不会于父级 RecyclerView 完全脱离。如果视图被认为是 **Dirty**，如果不需要重新 **Bingding** 或者通过 **Adapter** 修改，则不修改。
* **Dirty(View)**: 子视图在显示之前必须由 **Adapter** 复原。

### Positions in RecyclerView

RecyclerView 在 `RecyclerView.Adapter` 和 `RecyclerView.LayoutManager` 之间引入了一个额外的抽象级别，以便能够在布局计算期间批量检测数据集更改。这样可以节省 `LayoutManager` 以跟踪适配器的更改以计算动画。它也有助于性能，因为所有的视图绑定同时发生，避免了不必要的绑定。

因此，RecyclerView 中有两种类型的 **Position** 相关方法：

* layout position: Item 在最新布局中的 position。这是从 LayoutManager 的角度
* adapter position: Item 在 adapter 中的 position。这是从 Adapter 的角度

这两个位置是相同的，除了调度 `adapter.notify` 事件和计算更新的布局之间的时间。

方法返回或接收 *LayoutPositon* 使用 position 作为最新的布局计算(比如 `getPosition`,`findVIewHolderForLayoutPosition(int)`)。这些 position 包括所有更改，直到最后的布局计算。可以依靠这些 position 与用户在屏幕上看到的一致。

方法返回或接收 `AdapterPosition`(比如 `getAdapterPosition`，`findViewHolderForAdapterPosition(int)`)。当你需要使用最新的 adapter position 时，应该使用这些方法，即使它们可能还没被反映到布局中。比如，如果你想要在 adapter 中访问 ViewHolder 的点击。请注意，如果已调用 `notifyDataSetChanged()`，且还没计算布局，则这些方法可能无法计算 adapter position。在这种情况下，你应该小心处理 `NO_POSITION` 或 `null` 的结果。

### RecyclerView.Recycler

一个 Recycler 负责管理 **scrapped** 或 **detached** 的 item 视图以供重复使用。

**scrapped** 的视图是仍然 **attached** 到它的父 RecyclerView，但已被标记为删除或者重用。

Recycler 的典型用途将是获取 adapter 在给定的 position 或者 item ID 上的视图集合。如果要重复使用的视图被认定为 **dirty** ，则要 adapter 重新绑定。如果不是，则可以通过 LayoutManager 快速重用该视图。清理那些不需要 **requested layout** 的布局可能会被 LayoutManager 重新定位，而不需要重新测量。

### RecyclerView.State

包含有关当前 RecyclerView 状态的有用信息，如目标滚动位置或者视图焦点。State 对象也可以保留任意数据，由资源 ID 标识。

通常情况下，RecyclerView 组件需要互相传递信息。为了在组件之间提供良好的数据总线，RecyclerView 将相同的状态对象传递给组件回调，这些组件可以使用它来交换数据。

如果你实现自定义组件，则可以使用 State 的 put/get/remove 方法在组件之间传递数据，而无需管理其生命周期。