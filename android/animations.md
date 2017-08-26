

### View.GONE

当控件动画还在运行中，对控件调用 `setVisibility(View.GONE)` 会导致不可隐藏。

``` java
final int childIndex = getAndVerifyPreorderedIndex(childrenCount, i, customOrder);
            final View child = getAndVerifyPreorderedView(preorderedList, children, childIndex);
			//当 child.getAnimation() != null 时，依然会绘制视图
            if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
                more |= drawChild(canvas, child, drawingTime);
            }
```

