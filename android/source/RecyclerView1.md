RecyclerView 是用于向大型数据集提供有限窗口的灵活布局。

### 术语

* **Adapter**:  一个 `RecyclerView.Adapter` 的子类，用于管理提供代表数据集中单项数据的 view。
* **Position**: 在 **Adapter** 中单项数据的位置。
* **Index**: 在调用 `getChildAt(int)` 时使用的附加的 child view 的下标。与 **Position** 对比。
* **Binding**: 准备 child view 以显示对应于 **Adapter** 中的 **Position** 的数据的过程。
* **Recycle(View)**: 一个之前用于特定 **Adapter Position** 的 view 被放进缓存中，以便之后再次显示相同类型的数据时候使用。这可以通过跳过布局的 **inflation** 和构造来大幅提高性能。
* **Scrap(View)**: 在布局期间已进入暂时分离状态的 child view。**Scrap** 视图可能会重复使用，而不会完全脱离父   RecyclerView。如果不需要重新 binding，或者如果 view 被认为是 dirty，则由 adapter 修改。
* **Dirty(View)**: child view 在显示之前必须由 **Adapter** 复原。



### Positions in RecyclerView

RecyclerView 在 `RecyclerView.Adapter` 和 `RecyclerView.LayoutManager` 之间引入了一个额外的抽象级别，以便能够在布局计算期间批量检测数据集更改。这样可以节省 `LayoutManager` 以跟踪适配器的更改以计算动画。它也有助于性能，因为所有的 view bind 同时发生，避免了不必要的绑定。

因此，RecyclerView 中有两种类型的 **Position** 相关方法：

* layout position: Item 在最新布局中的 position。这是从 LayoutManager 的角度
* adapter position: Item 在 adapter 中的 position。这是从 Adapter 的角度

这两个位置是相同的，除了调度 `adapter.notify` 事件和计算更新的布局之间的时间。

可以通过（比如。`getLayoutPosition()`，`findViewHolderForLayoutPosition(int)`）方法获取或者接收 **LayoutPosition**，使用这些 position 计算最新的布局。这些 positions 包含所有到最新布局计算的所有变化。你可以依靠这些 positions 和当前用户在屏幕上看到的保持一致。例如，如果屏幕上有个列表，并且用户要求第五个元素，则应该使用这些方法，因为它们和用户所看到的保持一致。

可以通过（比如。`getAdapterPosition()`，`findViewHolerForAdapterPosition(int)`）方法获取或者接收另外一种 position，称为 **AdapterPosition**。当你需要使用最新的 adapter position 时，即使它们还没被反映到布局上，你应该使用这些方法。例如，如果你想要在 ViewHolder 单击中访问 adapter 中的 item 时，则应使用 `getAdapterPosition()`。请注意，如果 `notifyDataSetChanged()` 已被调用且尚未计算新布局，则这些方法可能无法计算适配器位置。出于这个原因，你应该小心处理这些方法的 `NO_POSITION` 或者 null。

当你编写 `LayoutManager` 你几乎总是使用 layout positions，而在编写 `Adapter` 时，你大概想使用 adapter positions。



### RecyclerView.Adapter

Adapter 的基类。

Adapters 提供从应用程序特定数据集到 RecyclerView 中显示的 view binding。



### RecylerView.ViewHolder

ViewHolder 描述了 item view 和它在 RecylerView 中的位置元数据。

RecyclerView.Adapter 的实现应该继承 ViewHolder，并且添加用于缓存潜在成本高昂的 findViewById（int）结果的字段。

虽然 RecyclerView.LayoutParams 属于 RecyclerView.LayoutManager，但 ViewHolder 属于 Adapter。Adapter 应该可以自由使用自己的 ViewHolder 实现来存储更容易地绑定 view 内容。实现应该假定单个 item view 会保存对 ViewHolder 实例的强引用，而且 RecyclerView 实例可能会强引用额外的屏幕外的 item view 来进行缓存。



### RecyclerView.RecycledViewPool

RecyclerViewPool 让你可以在多个 RecyclerViews 之间共享 views。

如果你想要跨越不同的 RecyclerViews 去 recycle view，创建 RecyclerViewPool 的实例，并且使用 `setRecyclerPool(RecyclerViewPool)`。当你没提供实例时，RecyclerView 自己自动创建一个实例。



### RecyclerView.Recycler

一个 Recycler 负责管理 **scrapped** 或 **detached** 的 item view 以供重复使用。

**scrapped** 的视图是仍然 **attached** 到它的父 RecyclerView，但已被标记为删除或者重用。

RecyclerView.LayoutManager 对 Recycler 的典型用法是从 adapter 的数据集中获取代表给定 position 或者 item id 的数据的 views。如果被复用 view 被认为是 dirty，则 adapter 将被要求重新 bind view。否则， view 可以被 LayoutManager 快速复用，而不需要额外的工作。没有请求 layout 的 clean views 可能被 LayoutManager 重新 positioned，而不需要重新测量。



### RecyclerView.State

包含有关当前 RecyclerView 状态的有用信息，如目标滚动位置或者视图焦点。State 对象也可以保留任意数据，由资源 ID 标识。

通常情况下，RecyclerView 组件需要互相传递信息。为了在组件之间提供良好的数据总线，RecyclerView 将相同的状态对象传递给组件回调，这些组件可以使用它来交换数据。

如果你实现自定义组件，则可以使用 State 的 put/get/remove 方法在组件之间传递数据，而无需管理其生命周期。



### RecyclerView.LayoutManager

LayoutManager 负责测量和定位 RecyclerView 中的 item view，并确定何时回收不再可见的 item view 的策略。通过更改 LayoutManager，可以使得 RecyclerView 实现标准的垂直滚动列表，均匀的网格，交错的网格，水平滚动等等。提供一些常用的 LayoutMananger 作为平常使用。

如果 LayoutManager 指定默认构造函数或者带有签名（Context,AttributeSet,int,int）的构造函数，则 RecyclerView 将在 inflated 时，实例化并且设置为 LayoutManager。常用的属性可以从 `getProperties(Context,AttributeSet,int,int)` 中获取。如果 LayoutManager 指定了以上两个构造函数，那么非默认构造函数将优先。

``` java
/**                                                                                           
 * Instantiate and set a LayoutManager, if specified in the attributes.                       
 */                                                                                           
private void createLayoutManager(Context context, String className, AttributeSet attrs,       
        int defStyleAttr, int defStyleRes) {                                                  
    if (className != null) {                                                                  
        className = className.trim();                                                         
        if (!className.isEmpty()) {                                                           
            className = getFullClassName(context, className);                                 
            try {                                                                             
                ClassLoader classLoader;                                                      
                if (isInEditMode()) {                                                         
                    // Stupid layoutlib cannot handle simple class loaders.                   
                    classLoader = this.getClass().getClassLoader();                           
                } else {                                                                      
                    classLoader = context.getClassLoader();                                   
                }                                                                             
                Class<? extends LayoutManager> layoutManagerClass =                           
                        classLoader.loadClass(className).asSubclass(LayoutManager.class);     
                Constructor<? extends LayoutManager> constructor;                             
                Object[] constructorArgs = null;                                              
                try {
                  	//优先反射带参数的构造方法
                    constructor = layoutManagerClass                                          
                            .getConstructor(LAYOUT_MANAGER_CONSTRUCTOR_SIGNATURE);            
                    constructorArgs = new Object[]{context, attrs, defStyleAttr, defStyleRes};
                } catch (NoSuchMethodException e) {                                           
                    try {                                                                     
                      	//如果带参数的构造方法不存在，使用默认构造方法。
                        constructor = layoutManagerClass.getConstructor();                    
                    } catch (NoSuchMethodException e1) {                                      
                        e1.initCause(e);                                                      
                        throw new IllegalStateException(attrs.getPositionDescription()        
                                + ": Error creating LayoutManager " + className, e1);         
                    }                                                                         
                }                                                                             
                constructor.setAccessible(true);                                              
                setLayoutManager(constructor.newInstance(constructorArgs));                   
            } catch (ClassNotFoundException e) {                                              
                throw new IllegalStateException(attrs.getPositionDescription()                
                        + ": Unable to find LayoutManager " + className, e);                  
            } catch (InvocationTargetException e) {                                           
                throw new IllegalStateException(attrs.getPositionDescription()                
                        + ": Could not instantiate the LayoutManager: " + className, e);      
            } catch (InstantiationException e) {                                              
                throw new IllegalStateException(attrs.getPositionDescription()                
                        + ": Could not instantiate the LayoutManager: " + className, e);      
            } catch (IllegalAccessException e) {                                              
                throw new IllegalStateException(attrs.getPositionDescription()                
                        + ": Cannot access non-public constructor " + className, e);          
            } catch (ClassCastException e) {                                                  
                throw new IllegalStateException(attrs.getPositionDescription()                
                        + ": Class is not a LayoutManager " + className, e);                  
            }                                                                                 
        }                                                                                     
    }                                                                                         
}                                                                                             
```



### RecyclerView.ItemDecoration

ItemDecoration 允许应用添加一个特定的绘图和布局，从 adapter 的数据集偏移到特定的 item view。这对于绘制 items 之间的分隔符，高亮，分组等等非常有用。



### RecyclerView.SmoothScroller

平滑滚动的基类。处理基础的目标 view position 的跟踪，并且提供方法去触发程序性的滚动。



### RecyclerView.ItemAnimator

这个类定义了在对 Adapter 进行更改时，发生在 items 上的动画。

ItemAnimator 的子类可以对 ViewHolder 上的操作，实现自定义的动画。RecyclerView 将在动画开始时，管理保留这些 items，不过实现者必须在动画结束时，调用 `dispatchAnimationFinished(ViewHolder)`。换句话说，在每次 `animateAppearance(ViewHolder,ItemHolderInfo,ItemHolderInfo)`，`animateChange(ViewHolder,ViewHolder,ItemHolderInfo,ItemHolder)`，`animatePersistence(ViewHolder,ItemHolderInfo,ItemHolderInfo)`，和 `animateDisappearance(ViewHolder,ItemHolderInfo,ItemHolderInfo)` 调用 `dispatchAnimationFinished()`。

默认情况下，RecyclerView 将使用 `DefaultItemAnimator`。