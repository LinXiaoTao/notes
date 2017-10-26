* FloatingActionButton 默认 Behavior 会对 `AppBarLayout` 进行特殊处理：

  ``` java
  		@Override
          public boolean onDependentViewChanged(CoordinatorLayout parent, FloatingActionButton child,
                  View dependency) {
              if (dependency instanceof AppBarLayout) {
                  // If we're depending on an AppBarLayout we will show/hide it automatically
                  // if the FAB is anchored to the AppBarLayout
                	//当设置了 AnchoredView 时，会调用 onDependentViewChanged()，当 AnchoredView 为 AppBarLayout 时，进行自动显示和隐藏处理
                  updateFabVisibilityForAppBarLayout(parent, (AppBarLayout) dependency, child);
              } else if (isBottomSheet(dependency)) {
                  updateFabVisibilityForBottomSheet(dependency, child);
              }
              return false;
          }
  ```

  ​

