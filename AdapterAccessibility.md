> &emsp : 空中文宽度 </br>
> &ensp : 半个中文宽度</br>
> &nbsp ： 物理空格</br>
#### 适配无障碍
---
创建(Init)并初始化(Populate)节点，当用户与屏幕交互的时候会创建和初始化事件，并最终将事件发送给Parent(可以选择为事件添加信息或者将事件拦截)，最终由RootViewImpl发送给AccessibilityService，然后做出语音播报 
##### Real节点
---
增加**ContentDescription**属性，就可以实现</br>
[option]重写**onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info)** 填充必要的信息</br>
[option]重写**onInitializeAccessibilityEvent(AccessibilityEvent event)** 填充必要的信息</br>

**Note** : **ViewGroup boolean onRequestSendAccessibilityEvent(View child, AccessibilityEvent event)**</br>
Parent用来填充事件信息或者拦截事件的</br>
**View. announceForAccessibility(String text)** 发送事件，语音播报text</br>
**View.performAccessibilityActionInternal(action, arguments);** 用来处理action
```java
@Override
    public boolean onRequestSendAccessibilityEvent(View child, AccessibilityEvent event) {
        // Add a record for ourselves as well.
        AccessibilityEvent record = AccessibilityEvent.obtain();
        super.onInitializeAccessibilityEvent(record);

        int priority = (Integer) child.getTag();
        String priorityStr = "Priority: " + priority;
        record.setContentDescription(priorityStr);
        event.appendRecord(record);		//为事件添加信息
        return true;		//不拦截
    }
```
##### 虚拟节点
---
初始化 **ExploreByTouchHelper**
```java
	class MyCustomView extends View {
	     private MyVirtualViewHelper mVirtualViewHelper;

	     public MyCustomView(Context context, ...) {
	         ...
	         mVirtualViewHelper = new MyVirtualViewHelper(this);
	         ViewCompat.setAccessibilityDelegate(this, mVirtualViewHelper);
	     }

	     @Override
	     public boolean dispatchHoverEvent(MotionEvent event) {
	       return mHelper.dispatchHoverEvent(this, event)
	           || super.dispatchHoverEvent(event);
	     }

	     @Override
	     public boolean dispatchKeyEvent(KeyEvent event) {
	       return mHelper.dispatchKeyEvent(event)
	           || super.dispatchKeyEvent(event);
	     }

	     @Override
	     public boolean onFocusChanged(boolean gainFocus, int direction,
	         Rect previouslyFocusedRect) {
	       super.onFocusChanged(gainFocus, direction, previouslyFocusedRect);
	       mHelper.onFocusChanged(gainFocus, direction, previouslyFocusedRect);
	     }
	 }
	 mAccessHelper = new MyExploreByTouchHelper(someView);
	 ViewCompat.setAccessibilityDelegate(someView, mAccessHelper);
```

处理 **ExploreByTouchHelper**
```java
protected class TouchHelper extends ExploreByTouchHelper {

        public TouchHelper(View host) {
            super(host);
        }
        //获取VirtualNode 的id
        @Override
        protected int getVirtualViewAt(float x, float y) {
            Rect rect = mImage.getBounds();
            if (rect.contains((int)x, (int)y)) {
                return 0;
            }
            return INVALID_ID;
        }
        //添加所有virtualNode的id
        @Override
        protected void getVisibleVirtualViews(IntArray virtualViewIds) {
            virtualViewIds.add(0);
        }
        //填充事件(如果node没有填充ContentDescription，那么就会使用到event的ContentDescription)
        @Override
        protected void onPopulateEventForVirtualView(int virtualViewId, AccessibilityEvent event) {
            event.setContentDescription(getItemDescription());
        }
        //填充node信息
        @Override
        protected void onPopulateNodeForVirtualView(int virtualViewId, AccessibilityNodeInfo node) {
            node.setContentDescription(getItemDescription());//不能为null
            node.setBoundsInParent(mImage.getBounds());//设置范围
            node.addAction(AccessibilityNodeInfo.ACTION_CLICK);//如果virtualNode可以点击，必须添加这条信息
        }
        //处理点击事件
        @Override
        protected boolean onPerformActionForVirtualView(int virtualViewId, int action, Bundle arguments) {
            switch (action) {
                case AccessibilityNodeInfo.ACTION_CLICK:
                	//处理完了需要添加这条语句,告知
                    sendEventForVirtualView(virtualViewId, AccessibilityEvent.TYPE_VIEW_CLICKED);
                    return true;
            }
            return false;
        }
```

