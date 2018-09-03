### RelativeLayout分析

##### onMeasure
##### 一 排序
简单来说：就是根据关系排序，优先将关联最少的child放在队列开头，直到将所有child放入再队列中
##### 二 测量 (水平) 
1. 初始化
```JAVA
	for (int i = 0; i < count; i++) {
            View child = views[i];
            if (child.getVisibility() != GONE) {
                LayoutParams params = (LayoutParams) child.getLayoutParams();
                int[] rules = params.getRules(layoutDirection);

                applyHorizontalSizeRules(params, myWidth, rules);
                measureChildHorizontal(child, params, myWidth, myHeight);

                if (positionChildHorizontal(child, params, myWidth, isWrapContentWidth)) {
                    offsetHorizontalAxis = true;		//该标记服务于设置了CENTER_IN_PARENT或者CENTER_HORIZONTAL的child
                }
            }
        }
```

2. applyHorizontalSizeRules(params, myWidth, rules); 

	**负责填写childParams.mRight 和 childParams.mLeft两个属性，默认设置为VALUE_NOT_SET=Integer.MIN_VALUE**
```java
private void applyHorizontalSizeRules(LayoutParams childParams, int myWidth, int[] rules) {
        RelativeLayout.LayoutParams anchorParams;

        // VALUE_NOT_SET indicates a "soft requirement" in that direction. For example:
        // left=10, right=VALUE_NOT_SET means the view must start at 10, but can go as far as it
        // wants to the right
        // left=VALUE_NOT_SET, right=10 means the view must end at 10, but can go as far as it
        // wants to the left
        // left=10, right=20 means the left and right ends are both fixed
        childParams.mLeft = VALUE_NOT_SET;
        childParams.mRight = VALUE_NOT_SET;

        anchorParams = getRelatedViewParams(rules, LEFT_OF);		//找到LEFT_OF的View的LayoutParams
        if (anchorParams != null) {
        	//找到就填写childParams.mRight。
            childParams.mRight = anchorParams.mLeft - (anchorParams.leftMargin +
                    childParams.rightMargin);
        } else if (childParams.alignWithParent && rules[LEFT_OF] != 0) {
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }

        anchorParams = getRelatedViewParams(rules, RIGHT_OF);		//处理RIGHT_OF
        if (anchorParams != null) {
            childParams.mLeft = anchorParams.mRight + (anchorParams.rightMargin +
                    childParams.leftMargin);
        } else if (childParams.alignWithParent && rules[RIGHT_OF] != 0) {
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }

        anchorParams = getRelatedViewParams(rules, ALIGN_LEFT);			//处理ALIGN_LEFT
        if (anchorParams != null) {
            childParams.mLeft = anchorParams.mLeft + childParams.leftMargin;
        } else if (childParams.alignWithParent && rules[ALIGN_LEFT] != 0) {
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }

        anchorParams = getRelatedViewParams(rules, ALIGN_RIGHT);		//处理ALIGN_RIGHT
        if (anchorParams != null) {
            childParams.mRight = anchorParams.mRight - childParams.rightMargin;
        } else if (childParams.alignWithParent && rules[ALIGN_RIGHT] != 0) {
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }

        if (0 != rules[ALIGN_PARENT_LEFT]) {						//处理ALIGN_PARENT_LEFT
            childParams.mLeft = mPaddingLeft + childParams.leftMargin;
        }

        if (0 != rules[ALIGN_PARENT_RIGHT]) {						//处理ALIGN_PARENT_RIGHT
            if (myWidth >= 0) {
                childParams.mRight = myWidth - mPaddingRight - childParams.rightMargin;
            }
        }
    }

    //在getRelatedViewParams方法中调用，返回不为Gone的View
    private View getRelatedView(int[] rules, int relation) {
        int id = rules[relation];
        if (id != 0) {
            DependencyGraph.Node node = mGraph.mKeyNodes.get(id);
            if (node == null) return null;
            View v = node.view;

            // Find the first non-GONE view up the chain
            while (v.getVisibility() == View.GONE) {
                rules = ((LayoutParams) v.getLayoutParams()).getRules(v.getLayoutDirection());
                node = mGraph.mKeyNodes.get((rules[relation]));
                // ignore self dependency. for more info look in git commit: da3003
                if (node == null || v == node.view) return null;
                v = node.view;
            }

            return v;
        }

        return null;
    }

```
	**tips：如果关联的View为Gone，Layout就会根据Gone的View的关联，继续找，直到找到第一个不为Gone的View**

3. measureChildHorizontal(child, params, myWidth, myHeight);
	
	**负责测量child，主要根据上个函数得到的childParams.mRight 和 childParams.mLeft来获取ChildMeasureSpec**
```java
	private void measureChildHorizontal(
            View child, LayoutParams params, int myWidth, int myHeight) {
		//根据参数获取childWidthMeasureSpec
        final int childWidthMeasureSpec = getChildMeasureSpec(params.mLeft, params.mRight,
                params.width, params.leftMargin, params.rightMargin, mPaddingLeft, mPaddingRight,
                myWidth);

        //获取childHeightMeasureSpec
        //...省略部分代码
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

    //主要逻辑，这里的mySize是RelativeLayout中WidthMeasureSpec获取到的(最大值)
    private int getChildMeasureSpec(int childStart, int childEnd,
            int childSize, int startMargin, int endMargin, int startPadding,
            int endPadding, int mySize) {
        int childSpecMode = 0;
        int childSpecSize = 0;
        //...省略部分代码

        // Figure out start and end bounds.
        int tempStart = childStart;
        int tempEnd = childEnd;

        // If the view did not express a layout constraint for an edge, use
        // view's margins and our padding
        //如果没有childParams.mRight 或者 childParams.mLeft没有设置，就设置成最大状态下的值
        if (tempStart == VALUE_NOT_SET) {
            tempStart = startPadding + startMargin;
        }
        if (tempEnd == VALUE_NOT_SET) {
            tempEnd = mySize - endPadding - endMargin;
        }

        // Figure out maximum size available to this view
        final int maxAvailable = tempEnd - tempStart;				//最大可用空间

		//如果params.mRight和params.mLeft都设置过了，则确定了ChildMeasureSpec
		//比如左右都有依赖，那么该View如果设置为wrap_content，或者match_parent是失效的，此时宽度已经确定所以为MeasureSpec.EXACTLY
        if (childStart != VALUE_NOT_SET && childEnd != VALUE_NOT_SET) {		
            // Constraints fixed both edges, so child must be an exact size.
            childSpecMode = isUnspecified ? MeasureSpec.UNSPECIFIED : MeasureSpec.EXACTLY;
            childSpecSize = Math.max(0, maxAvailable);				
        } else {	//没有设置的情况,根据params.width判断确定ChildMeasureSpec
            if (childSize >= 0) {
                // Child wanted an exact size. Give as much as possible.
                childSpecMode = MeasureSpec.EXACTLY;

                if (maxAvailable >= 0) {
                    // We have a maximum size in this dimension.
                    childSpecSize = Math.min(maxAvailable, childSize);
                } else {
                    // We can grow in this dimension.
                    childSpecSize = childSize;
                }
            } else if (childSize == LayoutParams.MATCH_PARENT) {
                // Child wanted to be as big as possible. Give all available
                // space.
                childSpecMode = isUnspecified ? MeasureSpec.UNSPECIFIED : MeasureSpec.EXACTLY;
                childSpecSize = Math.max(0, maxAvailable);
            } else if (childSize == LayoutParams.WRAP_CONTENT) {
                // Child wants to wrap content. Use AT_MOST to communicate
                // available space if we know our max size.
                if (maxAvailable >= 0) {
                    // We have a maximum size in this dimension.
                    childSpecMode = MeasureSpec.AT_MOST;
                    childSpecSize = maxAvailable;
                } else {
                    // We can grow in this dimension. Child can be as big as it
                    // wants.
                    childSpecMode = MeasureSpec.UNSPECIFIED;
                    childSpecSize = 0;
                }
            }
        }

        return MeasureSpec.makeMeasureSpec(childSpecSize, childSpecMode);
    }
```
	**tips： 如果左右都有依赖，那么该View设置为wrap_content或者match_parent都失效，RetiveLayout依赖关系放在首位**

4. positionChildHorizontal(child, params, myWidth, isWrapContentWidth)
	
	**补全childParams.mRight和childParams.mLeft，因为有的View只依赖了一边，或者都没有依赖，所以会根据测量得到的宽度来确定childParams.mRight和childParams.mLeft**
```java
private boolean positionChildHorizontal(View child, LayoutParams params, int myWidth,
            boolean wrapContent) {

        final int layoutDirection = getLayoutDirection();
        int[] rules = params.getRules(layoutDirection);

        if (params.mLeft == VALUE_NOT_SET && params.mRight != VALUE_NOT_SET) {		
            // Right is fixed, but left varies
            params.mLeft = params.mRight - child.getMeasuredWidth();
        } else if (params.mLeft != VALUE_NOT_SET && params.mRight == VALUE_NOT_SET) {
            // Left is fixed, but right varies
            params.mRight = params.mLeft + child.getMeasuredWidth();
        } else if (params.mLeft == VALUE_NOT_SET && params.mRight == VALUE_NOT_SET) {	//如果左右都不确定
            // Both left and right vary
            if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_HORIZONTAL] != 0) {	//如果设置了CENTER_IN_PARENT或者CENTER_HORIZONTAL
                if (!wrapContent) {			//如果RelativeLayout的宽度是确定的
                    centerHorizontal(child, params, myWidth);	// 那么放在中间	
                } else {					//如果宽度不确定
                    positionAtEdge(child, params, myWidth);// 暂时放在左上角，后面当RelativeLayout确定宽度的时候会调整位置(类似FrameLayout)
                }
                return true;								//返回赋值给offsetHorizontalAxis，需要特殊处理
            } else {
                // This is the default case. For RTL we start from the right and for LTR we start
                // from the left. This will give LEFT/TOP for LTR and RIGHT/TOP for RTL.
                //如果两边都没有依赖，且没有设置CENTER_IN_PARENT或者CENTER_HORIZONTAL，那么就放在最左上角
                positionAtEdge(child, params, myWidth);
            }
        }
        return rules[ALIGN_PARENT_END] != 0;
    }
```
	**tips：child两边都无依赖且含有CENTER_HORIZONTAL或者CENTER_HORIZONTAL属性时，后续当RelativeLayout确定宽高的时候会被调整位置(类似FrameLayout)**

5. 垂直方向上还有还会做一些判断
```java
	//因为此时child的基本参数都填写完全，所以这个时候会比较每个child的宽度，来确定RelativeLayout的最终大小
	if (isWrapContentWidth) {
            if (isLayoutRtl()) {
                if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                    width = Math.max(width, myWidth - params.mLeft);
                } else {
                    width = Math.max(width, myWidth - params.mLeft + params.leftMargin);
                }
            } else {
                if (targetSdkVersion < Build.VERSION_CODES.KITKAT) {
                    width = Math.max(width, params.mRight);
                } else {
                    width = Math.max(width, params.mRight + params.rightMargin);//比较每个view的有边界，取最大值作为RelativeLayout的宽度
                }
            }
        }
```

##### 三. 调整

```JAVA
	if (isWrapContentWidth) {
            // Width already has left padding in it since it was calculated by looking at
            // the right of each child view
            width += mPaddingRight;

            if (mLayoutParams != null && mLayoutParams.width >= 0) {
                width = Math.max(width, mLayoutParams.width);
            }

            width = Math.max(width, getSuggestedMinimumWidth());
            width = resolveSize(width, widthMeasureSpec);
			
			//当RelativeLayout的确定宽度后，那么设置了的CENTER_IN_PARENT或者CENTER_HORIZONTAL的Child会重新编排位置
			//offsetHorizontalAxis为true的条件是有至少一个child两边都无依赖且含有CENTER_HORIZONTAL或者CENTER_HORIZONTAL属性
            if (offsetHorizontalAxis) { 
                for (int i = 0; i < count; i++) {
                    final View child = views[i];
                    if (child.getVisibility() != GONE) {
                        final LayoutParams params = (LayoutParams) child.getLayoutParams();
                        final int[] rules = params.getRules(layoutDirection);
                        //修正含有CENTER_HORIZONTAL或者CENTER_HORIZONTAL属性的child
                        if (rules[CENTER_IN_PARENT] != 0 || rules[CENTER_HORIZONTAL] != 0) {
                            centerHorizontal(child, params, width);
                        } else if (rules[ALIGN_PARENT_RIGHT] != 0) {
                            final int childWidth = child.getMeasuredWidth();
                            params.mLeft = width - mPaddingRight - childWidth;
                            params.mRight = params.mLeft + childWidth;
                        }
                    }
                }
            }
        }
```

**总结**
> 1. RelativeLayout先处理关联关系，并初步确定child的位置，最后再处理并调整CENTER_HORIZONTAL或者CENTER_HORIZONTAL属性的child
> 2. 如果左右都有依赖，那么该View设置为wrap_content或者match_parent都失效，RetiveLayout依赖关系放在首位
> 3. RelativeLayout会尽可能满足child的需求，即使自己是wrap_content，当child为match_parent，那么RelativeLayout也会占满父类，来尽可能满足child
> 4. 调整位置的前提条件是至少一个child两边都无依赖且含有CENTER_HORIZONTAL或者CENTER_HORIZONTAL属性，且全部child都回有条件调整