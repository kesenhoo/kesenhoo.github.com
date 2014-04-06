---
layout: post
title: "Android Notes(06) - Touch事件分发响应机制"
date: 2014-04-05 13:13
comments: true
sidebar: false
categories: Android Android:Notes
---

## 1)概述
Android的TouchEvent通常包含三个动作,ACTION_DOWN,ACTION_MOVE与ACTION_UP。发出的顺序是DOWN->MOVE->MOVE->...->UP(注意MOVE事件是否能够被触发取决于操作手势里面是否包含了移动的动作)。

消息分发流程，从上到下，从父到子：Activity->ViewGroup1->ViewGroup1的子ViewGroup2->...->Target View  
消息响应流程，从下到上，从子到父：Target View->...->ViewGroup1的子ViewGroup2->ViewGroup1->Activity

- **public boolean dispatchTouchEvent(MotionEvent ev);**  
事件分发处理函数，通常会在Activity层根据UI的显示情况，把事件传递给相应的ViewGroup。下面的演示代码中，为了方便模拟，会直接return true，解说中称之为“丢弃”。(因为事件实际上没有传递给任何组件，没有被消费，而且是主动的行为，故称之为丢弃)

- **public boolean onInterceptTouchEvent(MotionEvent ev);**  
对分发的事件进行拦截，注意拦截ACION_DOWN与其他ACTION的差异。  
第1种情况：如果ACTION_DOWN的事件没有被拦截，顺利找到了TargetView，那么后续的MOVE与UP都能够下发。如果后续的MOVE与UP下发时还有继续拦截的话，事件只能传递到拦截层，并且发出ACTION_CANCEL。  
第2种情况：如果ACITON_DOWN的事件下发时被拦截，导致没有找到TargetView，那么后续的MOVE与UP都无法向下派发了，在Activity层就终止了传递。

- **public boolean onTouchEvent(MotionEvent ev);**  
响应处理函数,如果有设置对应listener的话,这里还会与onTouch,onClick,onLongClick有关联。具体执行顺序是onTouch()->onTouchEvent()->onClick()->onLongClick()。是否能够顺序执行，取决于每个方法的返回值是true还是false。具体这里不展开说。

**强关注点：dispatch与intercept的差异，ACTION_DOWN与其他ACITON会对寻找target组件带来差异，而是否寻找到Target组件对整个流程有着重大的的影响。** 
	
<!-- More -->

## 2)dispatchTouchEvent()的源码解读
```java Android 4.4 ViewGroup.dispatchTouchEvent()源码解读
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    ......
    boolean handled = false;
    ......
    final int action = ev.getAction();
    final int actionMasked = action & MotionEvent.ACTION_MASK;

    // 1)处理初始的ACTION_DOWN
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // 把ACTION_DOWN作为一个Touch手势的始点，清除之前的手势状态。
        cancelAndClearTouchTargets(ev); //清除前一个手势，*关键操作:mFirstTouchTarget重置为null*
        resetTouchState(); //重置Touch状态标识
    }

    // 2)检查是否会被拦截
    final boolean intercepted;
    if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
        // 是ACTION_DOWN的事件，或者mFirstTouchTarget不为null(已经找到能够接收touch事件的目标组件)
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0; 
        // 判断禁止拦截的FLAG，因为requestDisallowInterceptTouchEvent(boolean disallowIntercept)方法可以禁止执行是否需要拦截的判断
        if (!disallowIntercept) {
            // 禁止拦截的FLAG为false，说明可以执行拦截判断，则执行此ViewGroup的onInterceptTouchEvent方法
            intercepted = onInterceptTouchEvent(ev); // 此方法默认返回false，如果想修改默认的行为，需要override此方法，修改返回值。
            ev.setAction(action);
        } else {
            // 禁止拦截的FLAG为ture，说明没有必要去执行是否需要拦截了，这个事件是无法拦截的，能够顺利通过，所以设置拦截变量为false
            intercepted = false; 
        }
    } else {
        // 当不是ACTION_DOWN事件并且mFirstTouchTarget为null(意味着没有touch的目标组件)时，这个ViewGroup应该继续执行拦截的操作。
        intercepted = true;
    }
    // 通过前面的逻辑处理，得到了是否需要进行拦截的变量值

    final boolean canceled = resetCancelNextUpFlag(this) || actionMasked == MotionEvent.ACTION_CANCEL;
    final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
    TouchTarget newTouchTarget = null;
    boolean alreadyDispatchedToNewTouchTarget = false;
    if (!canceled && !intercepted) {
        // 不是ACTION_CANCEL并且拦截变量为false
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // 在ACTION_DOWN时去寻找这次DOWN事件新出现的TouchTarget
            final int actionIndex = ev.getActionIndex(); // always 0 for down

            .....
            
            final int childrenCount = mChildrenCount;
            if (newTouchTarget == null && childrenCount != 0) {
                // 根据触摸的坐标寻找能够接收这个事件的子组件
                final float x = ev.getX(actionIndex);
                final float y = ev.getY(actionIndex);
                
                final View[] children = mChildren;
                // 逆序遍历所有子组件
                for (int i = childrenCount - 1; i >= 0; i--) {
                    final int childIndex = i;
                    final View child = children[childIndex];
                    // 寻找可接收这个事件并且组件区域内包含点击坐标的子View
                    if (!canViewReceivePointerEvents(child) || !isTransformedTouchPointInView(x, y, child, null)) {
                        continue;
                    }

                    newTouchTarget = getTouchTarget(child); // 找到了符合条件的子组件，赋值给newTouchTarget
                    
                    ......

                    // 把ACTION_DOWN事件传递给子组件进行处理
                    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                        // 如果此子ViewGroup消费了这个touch事件
                        mLastTouchDownTime = ev.getDownTime();
                        mLastTouchDownIndex = childIndex;
                        mLastTouchDownX = ev.getX();
                        mLastTouchDownY = ev.getY();
                        // 则为mFirstTouchTarget赋值为newTouchTarget，此子组件成为新的touch事件的起点
                        newTouchTarget = addTouchTarget(child, idBitsToAssign); 
                        alreadyDispatchedToNewTouchTarget = true;
                        break;
                    }
                }
            }
            ......
        }
    }

    // 经过前面的ACTION_DOWN的处理，有两种情况。
    if (mFirstTouchTarget == null) {
        // 情况1：(mFirstTouchTarget为null) 没有找到能够消费touch事件的子组件或者是touch事件被拦截了，
        // 那么在ViewGroup的dispatchTransformedTouchEvent方法里面，处理Touch事件则和普通View一样，
        // 自己无法消费，调用super.dispatchOnTouchEvent()把事件回递给父ViewGroup进行处理
        handled = dispatchTransformedTouchEvent(ev, canceled, null, TouchTarget.ALL_POINTER_IDS);
    } else {
        // 情况2：(mFirstTouchTarget!=null) 找到了能够消费touch事件的子组件，那么后续的touch事件都可以传递到子View
        TouchTarget target = mFirstTouchTarget;
        // (这里为了理解简单，省略了一个Target List的概念，有需要的同学再查看源码)
        while (target != null) {
            if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                // 如果前面利用ACTION_DOWN事件寻找符合接收条件的子组件的同时消费掉了ACTION_DOWN事件，这里直接返回true
                handled = true;
            } else {
                final boolean cancelChild = resetCancelNextUpFlag(target.child) || intercepted;
                // 对于非ACTION_DOWN事件，则继续传递给目标子组件进行处理(注意这里的非ACTION_DOWN事件已经不需要再判断是否拦截)
                if (dispatchTransformedTouchEvent(ev, cancelChild, target.child, target.pointerIdBits)) {
                    // 如果target子组件进行处理，符合某些条件的话，会传递ACTION_CANCEL给target子组件
                    // 条件是：如果ACTION_DOWN时没有被拦截，而后面的touch事件被拦截，则需要发送ACTION_CANCEL给target子组件
                    handled = true;
                }
                ......
            }
        }
    }

    if (canceled || actionMasked == MotionEvent.ACTION_UP) {
        // 如果是ACTION_CANCEL或者ACTION_UP，重置Touch状态标识，mFirstTouchTarget赋值为null，后面的Touch事件都无法派发给子View
        resetTouchState();
    }
    ......

    return handled;
}
```

## 3)dispatchTouchEvent()流程图
![dispatchtouchevent_process.jpg](/images/articles/dispatchtouchevent_process.jpg "演示ViewGroup中的dispatchTouchEvent的流程")

## 4)代码举例说明
![dispatchtouchevent_demo.jpg](/images/articles/dispatchtouchevent_demo.jpg "Demo的layout层级")

[Demo Source Code](https://github.com/kesenhoo/TouchEventDemo.git)下面是截取的片段
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_DOWN");
            break;
            //Log.i(TAG, "[dispatchTouchEvent] -> ACTION_DOWN, return true");
            //return true;
        case MotionEvent.ACTION_MOVE:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_MOVE");
            break;
            //Log.i(TAG, "[dispatchTouchEvent] -> ACTION_MOVE, return true");
            //return true;
        case MotionEvent.ACTION_UP:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_UP");
            break;
        case MotionEvent.ACTION_CANCEL:
            Log.d(TAG, "[dispatchTouchEvent] -> ACTION_CANCEL");
            break;
        default:
            break;
    }
    boolean superReturn = super.dispatchTouchEvent(ev);
    Log.i(TAG, "[dispatchTouchEvent] return super. = " + superReturn);
    return superReturn;
}
```
下面演示的每一种情况，操作均为点击中间的Button，然后松开。请仔细看下面的案例，里面均有对应的解释。

### Case 0:没有任何的分发丢弃，也没有任何的拦截
![Touch_Case_0_All_Normal.png](/images/articles/Touch_Case_0_All_Normal.png "CASE 0")

### Case 1:Activity层的dispatch函数对ACTION_DOWN进行return true.
![Touch_Case_1_Activity_Dispatch_Down_Return_True.png](/images/articles/Touch_Case_1_Activity_Dispatch_Down_Return_True.png "CASE 1")

### Case 2:ParentLayout层的dispatch函数对ACTION_DOWN进行return true.
![Touch_Case_2_Parent_Dispatch_Down_Return_True.png](/images/articles/Touch_Case_2_Parent_Dispatch_Down_Return_True.png "CASE 2")

因为ChildLayout层的dispatch函数对ACITON_DOWN进行return true和在activity层,ParentLayout层是类似的逻辑，因为都没有找到Target组件，又没有拦截的因素影响，所以后续的MOVE与UP都只传递到DOWN被return true的那一层截至，然后都回传，也都没有被消费掉。(注意发生在ChildLayout层的return true与ParentLayout层的差异在于：回传时，只有return层与activity层才可以接收到onTouchEvent()的回调，但是默认都无法消费)。

### Case 3:Activity层的dispatch函数对ACTION_MOVE进行return true.
![Touch_Case_2_Activity_Dispatch_Move_Return_True.png](/images/articles/Touch_Case_3_Activity_Dispatch_Move_Return_True.png "CASE 3")

因为ParentLayout层的dispatch函数对ACITON_MOVE进行return true和在activity层是类似的道理，不做新的分析

### Case 4:ParentLayout层的intercept函数对ACTION_DOWN进行return true.
![Touch_Case_4_Parent_Intercept_Down_Return_True.png](/images/articles/Touch_Case_4_Parent_Intercept_Down_Return_True.png "CASE 4")

### Case 5:ChildLayout层的intercept函数对ACTION_DOWN进行return true.
![Touch_Case_5_Child_Intercept_Down_Return_True.png](/images/articles/Touch_Case_5_Child_Intercept_Down_Return_True.png "CASE 5")

### Case 6:ChildLayout层的intercept函数对ACTION_MOVE进行return true.
![Touch_Case_6_Child_Intercept_Move_Return_True.png](/images/articles/Touch_Case_6_Child_Intercept_Move_Return_True.png "CASE 6")

## 5)写在最后
- 对于dispatch分发某个事件的情况：
	- 如果是ACTION_DOWN被return true,那么在哪一层return的，后续的MOVE与UP都只传递到该层，然后回传(**Case 1,2**)(注意在回传的过程中只有在return层与activity层才会触发onTouchEvent，中间若是有其他层，均会被跳过。这一规律暂时没有找到比较有力的解释，需要查看更多的源码。)
	- 如果非ACTION_DOWN被return true,意味着DOWN事件正常被下发并找到Target组件，那么后续只有被return的事件会无法正常下发，并只传递到return层，没有return的事件还能够正常下方到Target组件并被Target消费。(**Case 3**)
- 对于intercept拦截某个事件的情况：
	- 如果ACTION_DOWN被拦截，无论拦截发生在哪一层，都会导致Target组件都无法找到，那么后续的MOVE与UP事件都只在Activity层处理，不会下发(**Case 4,5**)。
	- 如果ACTION_DOWN没被拦截，此时可以找到Target组件，DOWN事件是正常被消费。后续的MOVE如果被拦截，会对子组件触发CANCEL的事件，并且UP事件只能传递到拦截MOVE的那一层，无法消费并返回(**Case 6**)。Ps:因为Case 6演示的是在ChildLayout层对MOVE进行拦截，所以看到的效果是Button直接收到了CANCEL,实际上如果是ParentLayout对MOVE进行拦截，那么CANCEL事件需要经过ChildLayout(如果有需要的话，可以在这里继续拦截CANCEL)，最终CANCEL事件都是由Button进行消费。

**经过上面的描述，对Android的Touch事件传递机制应该有更深入的了解，理解错误或者有偏差的地方，欢迎提出一起讨论，谢谢！**

