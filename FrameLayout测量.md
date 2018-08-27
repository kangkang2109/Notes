#####FrameLayout两次测量

#####准备阶段
```java
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
		int count = getChildCount();
		//判断是否自身的宽高是否是固定的(不等于MeasureSpec.EXACTLY)
	    final boolean measureMatchParentChildren =
	            MeasureSpec.getMode(widthMeasureSpec) != MeasureSpec.EXACTLY ||
	            MeasureSpec.getMode(heightMeasureSpec) != MeasureSpec.EXACTLY;
	    mMatchParentChildren.clear();				//ArrayList<View> 一个数组存子View

	    int maxHeight = 0;
	    int maxWidth = 0;
	    int childState = 0;				//状态表示父View是否满足子View的大小
```

#####第一次测量
```java
		//循环测量子View，获取一个最大的width和最大height
		for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            //mMeasureAllChildren是个标记位(表示是否是所有的View都要测量)，或者只是测量非GONE状态的View。
            if (mMeasureAllChildren || child.getVisibility() != GONE) {
            	//ViewGroup内部提供了两种方式来测量子View measureChild 和 measureChildWithMargins区别是考虑margin
                measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                maxWidth = Math.max(maxWidth,
                        child.getMeasuredWidth() + lp.leftMargin + lp.rightMargin);
                maxHeight = Math.max(maxHeight,
                        child.getMeasuredHeight() + lp.topMargin + lp.bottomMargin);
                //获取子View的状态
                childState = combineMeasuredStates(childState, child.getMeasuredState());
                //如果父View是wrap_content，子View是match_parent那么，子View的预估模式为AT_MOST，所以第一次测量没有限制大小
                //如果该FrameLayout宽高有一个不是确定值，那么第一次测量的所有子View对应的都是AT_MOST
                if (measureMatchParentChildren) {
                    if (lp.width == LayoutParams.MATCH_PARENT ||
                            lp.height == LayoutParams.MATCH_PARENT) {
                    	//如果子View是match_parent，后续需要重新测量。
                        mMatchParentChildren.add(child);
                    }
                }
            }
        }
        // Account for padding too
        maxWidth += getPaddingLeftWithForeground() + getPaddingRightWithForeground();
        maxHeight += getPaddingTopWithForeground() + getPaddingBottomWithForeground();

        // Check against our minimum height and width
        maxHeight = Math.max(maxHeight, getSuggestedMinimumHeight());
        maxWidth = Math.max(maxWidth, getSuggestedMinimumWidth());

        // Check against our foreground's minimum height and width
        final Drawable drawable = getForeground();
        if (drawable != null) {
            maxHeight = Math.max(maxHeight, drawable.getMinimumHeight());
            maxWidth = Math.max(maxWidth, drawable.getMinimumWidth());
        }
    	//确定FrameLayout的宽高
        setMeasuredDimension(resolveSizeAndState(maxWidth, widthMeasureSpec, childState),
                resolveSizeAndState(maxHeight, heightMeasureSpec,
                        childState << MEASURED_HEIGHT_STATE_SHIFT));

```
1. 循环遍历子View
2. 测量所有子View，获取到最大的width和height
3. 如果FrameLayout宽高不固定且子View为match_parent(measureMatchParentChildren为true)，则将childview加入数组。
4. 确定FrameLayout的宽高

>**之前FrameLayout未确定宽高，第一次测量完毕后确定了FrameLayout宽高，后续子View为match_parent的就可以重新测量得到正确的宽高**

#####第二次测量

```java
count = mMatchParentChildren.size();
//如果只有一个那么就不需要重新测量。
if (count > 1) {
    for (int i = 0; i < count; i++) {
        final View child = mMatchParentChildren.get(i);
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

        final int childWidthMeasureSpec;
        if (lp.width == LayoutParams.MATCH_PARENT) {
        	//如果未match_parent，重新创建childWidthMeasureSpec，建立再已经确定宽高的FrameLayout上
            final int width = Math.max(0, getMeasuredWidth()
                    - getPaddingLeftWithForeground() - getPaddingRightWithForeground()
                    - lp.leftMargin - lp.rightMargin);
            childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(
                    width, MeasureSpec.EXACTLY);
        } else {
            childWidthMeasureSpec = getChildMeasureSpec(widthMeasureSpec,
                    getPaddingLeftWithForeground() + getPaddingRightWithForeground() +
                    lp.leftMargin + lp.rightMargin,
                    lp.width);
        }

        final int childHeightMeasureSpec;
        if (lp.height == LayoutParams.MATCH_PARENT) {
            final int height = Math.max(0, getMeasuredHeight()
                    - getPaddingTopWithForeground() - getPaddingBottomWithForeground()
                    - lp.topMargin - lp.bottomMargin);
            childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(
                    height, MeasureSpec.EXACTLY);
        } else {
            childHeightMeasureSpec = getChildMeasureSpec(heightMeasureSpec,
                    getPaddingTopWithForeground() + getPaddingBottomWithForeground() +
                    lp.topMargin + lp.bottomMargin,
                    lp.height);
        }
        //重新测量
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

```

1. 如果只有一个那么就不需要重新测量，因为此时FrameLayout的宽高就是当前子View的宽高
2. 如果子View为LayoutParams.MATCH_PARENT，重新创建childWidthMeasureSpec，在确定宽高的FrameLayout的基础上。