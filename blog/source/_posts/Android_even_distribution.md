---
title: Android事件分发机制
date: 2017-05-03 16:43:57
tags:
   - Android
---
## 一.说些废话
Android事件分发真的非常非常重要，几乎所有的滑动冲突以及点击冲突都需要深刻了解该机制才可以解决问题。所以我希望大家可以仔细阅读该篇文章并且自己手动来实验，一定要自己打断点看看源码，不管干什么都要下功夫不是吗？    

虽然重要，但其实Android事件分发机制也很简单，只要理解了Android事件分发的三个方法，以及传递的流程，你就可以轻松掌握Android事件分发机制。    

在开始之前，再说一下事件流：    

事件流指的是一次完整的触摸事件，一次完整的触摸事件应该包括是:     

`down（一次按下）-->move move move（多次滑动）.....-->up（一次抬起）`      

所以事件流总是以down事件为开端，以up事件为终止。        

那么接下来就正式开始吧！        



## 二. 重要方法
首先看这三个方法的名称以及拥有情况：         

<font color='red'>注意：类型不同，方法名相同，源码并不一致</font>     
       

| 方法名                    | Activity   |  ViewGroup |  View     |
| ----------------------    | :----:     | :----:     | :----:   |
|  dispatchTouEvent         | true       |   true     |  true    |
| onInterceptTouchEvent     |   false    |   true     |  false   |
| onTouchEvent              |   true     |   true     |  true    |
       

true代表拥有，false代表没有此方法     

如果想要了解Android的事件分发机制，就必须先了解这三个方法      


- `dispatchTouEvent()`      

	事件的分发方法，用来分发事件，在Activity、ViewGroup、View中都有该方法。
- `onInterceptTouchEvent()`       

	事件的拦截方法，只有ViewGroup有该方法。
- `onTouchEvent()`      

	事件的响应,当该事件属于我时，会执行该方法，返回true代表消费事件，返回false代表不消费事件并将该事件向外传递。

三个方法的含义知道了，这很easy，没错，就三个方法而已。接下来让我们深入分析：       

![](http://7xvzby.com1.z0.glb.clouddn.com/back_I_begin_zb.jpg)
## 三. 具体案例
首先，我们来想象一个简单的布局             

![布局](http://7xvzby.com1.z0.glb.clouddn.com/%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91_0.png)              
        

这个布局很简单,FrameLayout1的中间放了一个FrameLayout2，FrameLayout2的中间放了一个Button。       

我在各个布局当中重写了所有事件响应的方法，并且没有修改任何返回值，接下来我要点击Button了！      

我点！      

![我点](http://7xvzby.com1.z0.glb.clouddn.com/%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91_1.png)     

首先接收到事件的是Activity,接下来事件传递到F1,F2,最后传递到Button的onTouchEvent被消费，很好理解，因为**事件由外向里传递**        
 
接下来我们就着这个简单的按下事件，来分析一源码         


### 1、Activity
#### (1) `dispatchTouchEvent()`
现在以debug的模式来看源码。      

我的源码是Android－23。        

首先接收到事件的是就是acitivity，所以我们把断点就放在Activity的`dispatchTouchEvent()`方法中       

![](http://7xvzby.com1.z0.glb.clouddn.com/%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91_2.png)      

然后debug模式运行项目，点击下一步进入Activity的`dispatchTouchEvent()`的源码中：      

![](http://7xvzby.com1.z0.glb.clouddn.com/%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91_3.png)      

上面的注释大概说的是：你可以重写此方法在所有窗体接收到事件之前将其拦截，如果不需要拦截，则保持原样。       

简单来说呢，就是：**我可以重写该方法修改返回值使整个Activity都无法响应事件。**       

如果我在Activity中重写该方法并修改了它的返回值，不管将该方法的返回值改为true或者false，该事件都消失，不会再向下传递（可怕）。        

**所以不要修改Activity的`dispatchTouchEvent()`的默认返回值。**         

接下来我们来分析上面的代码，首先是判断事件是否是down事件，如果是的话执行了`onUserInteraction()`。       

这个方法是个空实现，我们可以在Activity中重写该方法。这里你要想到的是，因为down事件是一个事件流的开端，并且这个方法放在了分发之前，在最上端，所以我们可以在`onUserInteraction()`中做一些事件响应开始的操作。          

接下来执行了Window的`superDispatchTouchEvent()`,点进去你会发现Window中的该方法是一个空实现，但是根据断点进去可以直接进到它的实现类：PhoneWindow（这也是断点的一个好处不是吗）。那么我们来看看PhoneWindow的：  
     
`superDispatchTouchEvent()`：
```
@Override
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```
代码就一行，调用了Decor类的`superDispatchTouchEvent()`       

但是!But!But,oh no!What's Decor?          

![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/%E6%83%A8%E4%B8%8D%E5%BF%8D%E7%9D%B9.png)       

[这里是一篇讲解Decor类的文章](http://www.jianshu.com/p/687010ccad66)，我也简单说一下：       

为什么Activity可以利用`setContentView()`来设置布局？谁是容器？         

没错，就是Decor。我们查看Decor会发现，它是PhoneWindow的内部类，继承自FrameLayout。         

FrameLayout？？难道我们`setContentView()`最终是调用了`Decor.addView()`吗？？      

这里我可以告诉你：虽然不是，但是原理就是这样的！        

看下面这张图片就会明白顶层布局的原理了：       

![](http://upload-images.jianshu.io/upload_images/1734948-cd6dd09d50cb0bb5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)      

titleBar就是标题栏，main.xml就是我们设置的布局文件，而最外层是Decor。      

好了，就到这里。对Android窗体感兴趣的小伙伴，快去看看[这篇讲解Decor类的文章](http://www.jianshu.com/p/687010ccad66)。        

我们接着查看源码，你会发现`mDecor.superDispatchTouchEvent(event)`最终调用的是ViewGroup的`dispatchTouchEvent()`。

```
/**
    Decor类的superDispatchTouchEvent
    注意Decor类继承自FrameLayout
    所以super指的即是FrameLayout
    但是FrameLayout没有重写该方法，查看源码会直接跳转至ViewGroup类
**/
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```
到这里，Activity已经成功将事件传递给ViewGroup，接下来将由ViewGroup来将事件一层一层传递至内部。

#### (2) `onTouchEvent()`
各位看官，请拉上去回头看Activity的`dispatchTouchEvent()`的最后一行：
`return onTouchEvent(ev)`
是这样的不？
也就是在`getWindow.superDispatchTouchEvent()`返回false时，将执行最后一行调用Activity的`onTouchEvent()`，所以Activity的`onTouchEvent()`的调用时机，是和接下来ViewGroup的事件分发有着密切关系的。
简单来说：只有当Activity下的所有View（ViewGroup）的`onTouchEvent()`都不消费事件时，才会调用Activity的`onTouchEvent()`。
### 2、ViewGroup
我们很顺利地来到了ViewGroup的`dispatchTouchEvent()`方法当中。
但是，在分析源码之前，我想让各位看官先明白，现在调用ViewGroup的`dispatchTouchEvent()`方法的变量，指的是谁？是我们的FrameLayou_ONE吗？
看官:"你他喵的484傻，Activity源码中明明写着`mDecor.superDispatchTouchEvent(event)`，很明显是Activity调用了Decor的`dispatchTouchEvent()`，这个ViewGroup肯定是Decor啊！能不能讲重点，这么多废话!“        

我：“.....”    
      
      
![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/%E9%80%80%E5%87%BA%E8%A3%85%E9%80%BC%E7%95%8C.jpg)         
行吧，提这个我是想再给各位看官强调一下，我们的layout文件仅仅是add到了系统顶层布局中，所以接收到事件的肯定是系统的顶层布局，而不是我们的layout中的布局。这个如何证实呢？             
各位看官可以在ViewGroup的`dispatchTouchEvent()`打个断点，然后看官就会发现，这个方法执行了很多次，每次都可以查看一下变量this，看看到底是什么？比如第一次的this：
```
com.android.internal.policy.PhoneWindow$DecorView{b9a23df V.E...... R....... 0,0-1080,1920}
```
好了，接下来我们就看看ViewGroup的事件分发机制吧!
#### (1)`dispatchTouchEvent()`
ViewGroup的`dispatchTouchEvent()`内容很多，有210行左右，有兴趣的同学可以打开ViewGroup去查看。我这里就按着步骤一部分一部分的贴出来。有些部分我也看不大懂，不过有很多的注释，阅读起来还是很方便的：
首先执行的是这个判断，判断是否分发事件，如果该方法返回false，则此次事件则会被丢弃，因为`dispatchTouchEvent()`的所有代码都包含在这个判断里！
```
if (onFilterTouchEventForSecurity(ev)){}
```
下面是`onFilterTouchEventForSecurity(ev)`这个判断的源码：
根据方法名和注释来理解，这是一个触摸事件安全过滤器。基本上这个方法都会返回true，不会发生系统丢弃事件的情况。
```
    /**
     * Filter the touch event to apply security policies.
     *
     * @param event The motion event to be filtered.
     * @return True if the event should be dispatched, false if the event should be dropped.
     *
     * @see #getFilterTouchesWhenObscured
     */
public boolean onFilterTouchEventForSecurity(MotionEvent event) {
    //noinspection RedundantIfStatement
    if ((mViewFlags & FILTER_TOUCHES_WHEN_OBSCURED) != 0&& (event.getFlags() & MotionEvent.FLAG_WINDOW_IS_OBSCURED) != 0) {
        // Window is obscured, drop this touch.
            return false;
    }
    return true;
}
```
第二件事情：如果是down事件（因为down事件是开端），清除上次事件流的处理结果和状态。
```
// Handle an initial down.
if (actionMasked == MotionEvent.ACTION_DOWN) {
    // Throw away all previous state when starting a new touch gesture.
    // The framework may have dropped the up or cancel event for the previous gesture
    // due to an app switch, ANR, or some other state change.
    cancelAndClearTouchTargets(ev);
    resetTouchState();
}
```
第三件事情：检查事件拦截是否拦截。
```
// Check for interception.
final boolean intercepted;
if (actionMasked == MotionEvent.ACTION_DOWN|| mFirstTouchTarget != null) {
    final boolean disallowIntercept = (mGroupFlags &FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
          intercepted = onInterceptTouchEvent(ev);
          ev.setAction(action); // restore action in case it was changed
    } else {
    intercepted = false;
    }
} else {
    // There are no touch targets and this action is not an initial down
    // so this view group continues to intercept touches.
    intercepted = true;
}
```
第四件事情：检查事件取消。
```
// Check for cancelation.
final boolean canceled = resetCancelNextUpFlag(this)|| actionMasked == MotionEvent.ACTION_CANCEL;
```

第五件事情：如果没有被拦截，也没有被取消，就将此次事件分发给自己的下一级：子ViewGroup或者子View。
这里的分发逻辑相当复杂，有兴趣的看官可以去自行阅读源码，或者参考[这篇文章](http://wangkuiwu.github.io/2015/01/04/TouchEvent-ViewGroup/)。
```
if (!canceled && !intercepted) {}
```
在最后，又进行一次检查取消标记，做了相应的处理:
```
// Update list of touch targets for pointer up or cancel, if needed.
if (canceled|| actionMasked == MotionEvent.ACTION_UP|| actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
    resetTouchState();
} 
else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
    final int actionIndex = ev.getActionIndex();
    final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
    removePointersFromTouchTargets(idBitsToRemove);
}
```
。

#### (2) `onInterceptTouchEvent()`
事件流的每个事件都是由外层向里层依次传递的，有时候我们希望虽然点击的是内部button，但是做出相应的是button的容器，而不是button，也就是说在事件传递到button之前，将事件拦截并且消费，这也就是事件拦截的作用。
事件拦截的方法源码很简单很简单：
```
public boolean onInterceptTouchEvent(MotionEvent ev) {
   return false;
}
```
默认不拦截事件，如果拦截事件将返回值改为true，事件将被拦截并交给拦截事件的ViewGroup的`onTouchEvent()`处理。
这个方法是ViewGroup独有的，难道Activity和View就无法拦截事件了吗？
首先，Activity的`dispatchTouchEvent()`的返回值不可轻易更改，更改之后整个Activity都无法响应事件，所以如果我们需要Activity做拦截操作，可以修改Activity的`dispatchTouchEvent()`的逻辑，使之满足某些条件时修改返回值，否则不做修改。这样就可以使Activity进行事件的拦截。
而至于View就更简单了，View本身就是最小的控件，事件传递到View是已经无法向下传递，所以无需拦截。
#### (3) `onTouchEvent()`
ViewGroup类中没有重写`onTouchEvent()`，由于ViewGroup继承了View，所以View和ViewGroup的`onTouchEvent()`完全一致。
这里我们再debug一下我们的demo：
将断点打到View的`onTouchEvent()`的第一行，然后点击button，会发现断点执行，而this指代的是我们的Button，而不是上层的ViewGroup，也就是说上层中根本没有执行`onTouchEvent()`，第一次执行`onTouchEvent()`时，事件已经分发到了View。
所以在这里，我们需要理解到这样一个地步：       

**事件从Activity的`dispatchTouchEvent()`开始，首先分发到ViewGroup（DecorView、我们自己的Layout布局文件等）的`dispatchTouchEvent()`，并且在每次分发的时候会调用`onInterceptTouchEvent()`判断事件是否被拦截，如果事件被拦截则会执行将事件拦截的ViewGroup的`onTouchEvent()`方法并将接下来的整个事件流，都交给自己来处理，不会重复执行`onInterceptTouchEvent()`，如果没有拦截事件，最终将事件成功传递给View，View将调用onTouchEvent()将事件消费。**    
    

上面这段话希望每位耐心看到这里的读者都能理解，在这里还需要解释一下的是ViewGroup拦截事件交给`onTouchEvent()`之后，如果`onTouchEvenet()`返回了false（ViewGroup的`onTouchEvent()`默认返回false），则代表不消费此事件，则此次事件将逐层向上传递，调用每一层的`onTouchEvnet()`，如果中途所有的`onTouchEvent()`都返回false，那么事件将最终传递到Activity的`onTouchEvent()`中，到这里事件完成整个传递过程（向里传递，里不要，再传出来），不管Activity的`onTouchEvent()`返回什么结果，事件都将消失。
### 3、View
#### (1)`dispatchTouchEvent()`
```
public boolean dispatchTouchEvent(MotionEvent event) {
       boolean result = false;
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }

        if (onFilterTouchEventForSecurity(event)) {
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
}
```
以上则是View的部分源码，我们只需要理解的是：在满足了相应条件之后，View的`dispatchTouchEvent()`主动调用了自己的`onTouchEvent()`，而View的`onTouchEvenet()`默认返回true消费事件。
#### (2)`onTouchEvent()`
`onTouchEvent`是用来消费事件的，在Activity、ViewGroup和View三者当中，只有View的`onTouchEvent()`默认返回true，代表消费事件。虽然ViewGroup和View的`onTouchEvent()`完全相同，但是其中存在某些逻辑判断，致使ViewGroup的返回值为false。
推荐做法还是将事件处理的逻辑放到`onTouchEvent()`当中，而分发和拦截只发挥自己应有的作用即可。
这里我就不贴View(ViewGroup)的`onTouchEvent()`的源码了，想看的同学可自行打开View搜索`onToucEvent()`。
## 四.总结
到这里，三者的各个方法的分析已经结束了，我想最后再简单总结一下：               

1、Activity的`dispatchTouchEvent()`不可修改返回值，否则将导致整个Activity都无法接收事件，不管修改后返回true或是false。    

2、在Activity中我们可以重写`onUserInteraction()`方法，来在整个事件流响应之前做自己想做的事情。    

3、一个事件的传递过程默认总是从Activity的`dispatchTouchEvent()`开始，到View的`onTouchEvent()`结束，注意，这里是默认。     

4、ViewGroup可以拦截事件交给自己的`onTouchEvent()`处理，如果自己的`onTouchEvent()`返回false，则事件不会消失，会继续向外层传递，如果一直没有被消费，那么事件传递到Activity的`onTouchEvent()`才会消失。      

5、ViewGroup的`onInterceptTouchEvent()`不会在每次事件都调用，整个事件流只会在down事件或是第一次接触到事件时调用，所以不要在该方法中做事件逻辑处理。       

6、三者的`onTouchEvent()`只有View的`onTouchEvent()`默认消费事件。       


------------
其实还有很多更深入的东西，这篇文章只是浅显地讲解了一下Android的事件分发机制，希望对各位看官有些帮助！

>有任何问题或疑问都可以联系我：mengyuanzz@126.com

