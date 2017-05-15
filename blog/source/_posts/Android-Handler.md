---
title: Android_Handler机制
date: 2017-05-15 14:57:07
tags:
---
Handler是Android消息通讯当中最常用的方式之一。
本篇小文将会从Handler的源码角度去浅析Handler。
## 总结 ##
因为Handler这个东西其实大家都会用，源码也多多少少地了解过，所以直接将最关键的话，写到前面，对源码感兴趣的看官可以在看完总结后再往下浏览源码：

1. 创建Handler
    ```
    Handler handler = new Handler();
    ```
 在主线程当中，可以直接进行Handler的创建。如果是在子线程当中，在创建之前必须先初始化Looper，否则会RuntimeException:
    ```
    Looper.prepare();
    Handler handler = new Handler();
    handler.sendEmptyMessage(8);
    Looper.loop();
    ```
 在初始化Looper的同时，一定要调用`Looper.loop()`来启动循环，否则Handler仍然无法正常接收。
 并且因为`Looper.loop()`有死循环的存在，`Looper.loop()`之后的代码将无法执行，所以需要将`Looper.loop()`放在代码的最后，详情可参考`ActivityThread`中的main方法创建Looper的流程。
 查看Handler的构造方法可以发现，其实Looper是Handler必要元素，但是在主线程初始化的时候，Looper已经初始化完成，所以无需再创建Looper，但是子线程的Looper需要我们自己初始化。
 并且Looper在每个线程只能存在一个，如果再去手动创建Looper也会抛出RuntimeException。

2. 发送Handler
handler的发送有很多方法，包括发送延时Handler、即时Handler、MessageHandler等等，但是查看这些发送方法的源码，就会发现这些发送Message的方法最终都会调用`MessageQueue.enqueueMessage()`方法。
这个方法其实就是将我们发送的Message入队到MessageQueue队列中，这样，我们的消息就已经发送成功，等待执行了。
3. 取出Handler
在`Looper.prepare()`的同时，总会执行`looper.loop()`语句与之对应。
查看`loop()`源码会发现，这个方法中有一个for(;;)的死循环，会无限执行`MessageQueue.next()`，而`MessageQueue`就是我们上一步将Meesage入队的对象。
也就是说在创建Looper时，就会启动`MessageQueue`的无限遍历。如果`MessageQueue`为空，`Looper.loop()`就会进入休眠，直到再有Message插入到`MessageQueue`中。
如果取到Message则会调用message.target.dispatchMessage()，将消息分发给对应的Handler。
4. 如何在`loop()`休眠之后唤醒`loop()`？
在Meesage入队的时候，也就是执行`MessageQueue.enqueueMessage()`方法时，`enqueueMessage()`有一个`nativeWeak()`的native方法，如果有消息进入，并且Looper是休眠状态，则会执行该方法唤醒Looper。
5. 整体流程
先调用`Looper.prepare()`创建Looper，在创建的同时会自动调用`Looper.loop()`执行死循环loop()。
死循环中会执行`MessageQueue.next()`方法去取出队列中的消息，当消息为空时，`MessageQueue.next()`方法中会执行`nativePollOnce()`的native方法休眠`Looper.loop()`死循环。
当有新的消息插入到`MessageQueue`中，也就是调用`MessageQueue.enqueueMessage()`方法，这个方法当中会判断Looper是否是休眠状态，如果是休眠状态会执行nativeWeak的native方法来唤醒Looper()。

## Handler的使用 ##

1. 主线程
在`主线程`当中，Handler可以作为一个成员变量直接进行创建：
    ```
    //注意，Handler属于android.os包
   private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            System.out.println("主线程的Handler");
        }
    };
    ```

 接着我们试着发送Handler：
    ```
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //发送一个空Handler，what为888，延迟3秒到达
        handler.sendEmptyMessageDelayed(888,3000);
    }
    ```
 发送Handler有很多方法：发送空Message、发送延迟的空Message、发送Message、发送延迟的Message等：
![](http://7xvzby.com1.z0.glb.clouddn.com/handler/handler_send_funcation.png)

 接着我们运行项目，就会发现3s后控制台有Log的打印。
2. 子线程
在`子线程`中创建Handler的流程，和在`主线程`基本一致，只是多了一步而已，但这一步非常关键。
我们先按照`主线程`的步骤，看看会有什么问题。
在`子线程`中创建Handler：
    ```
new Thread(){
    @Override
    public void run() {
        super.run();
        Handler handler = new Handler(){
             @Override
            public void handleMessage(Message msg) {
                System.out.println("子线程的Handler");
            }
        };
    }
}.start();
    ```
 接着我们运行项目，会发现项目崩溃了：
    ```
java.lang.RuntimeException: Can''t create handler inside thread that has not called Looper.prepare()
at android.os.Handler.<init>(Handler.java:200)
at android.os.Handler.<init>(Handler.java:114)
at com.my.oo.MainActivity$2$1.<init>(MainActivity.java:0)
    ```
 很明显，错误信息说的是**在当前线程中创建Handler失败因为没有执行Looper.prepare()**。
 那我们按照错误原因，在创建Handler之前，加上`Looper.prepare()`:
    ```
//子线程
new Thread(){
    @Override
    public void run() {
        super.run();
        //添加Looper.prepare()；
        Looper.prepare();
        //之后再创建Handler
        Handler handler = new Handler(){
             @Override
            public void handleMessage(Message msg) {
                System.out.println("子线程的Handler");
            }
        };
        //发送消息
        handler.sendEmptyMessage(111);
    }
}.start();
    ```
 这次再运行项目，我们就会发现项目正常运行没有问题。但是发送Handler仍然无法接收，那是因为我们没有启动Looper的遍历：
    ```
//子线程
new Thread(){
    @Override
    public void run() {
        super.run();
        //添加Looper.prepare()；
        Looper.prepare();
        //之后再创建Handler
        Handler handler = new Handler(){
             @Override
            public void handleMessage(Message msg) {
                System.out.println("子线程的Handler");
            }
        };
        //发送消息
        handler.sendEmptyMessage(111);
        //启动Looper的遍历功能
        Looper.loop();
    }
}.start();
    ```
 这里一定要注意，**`Looper.loop()`必须放到代码的最后。因为`Looper.loop()`中有死循环，会导致之后的代码无法执行**。这里可以等到查看`主线程`创建过程的源码时证实。
3. 使用总结
Handler的使用就是这么简单，要注意的就是`子线程`当中使用Handler时，一定要先调用`Looper.prepare()`，最后调用`Looper.loop()`，否则项目会崩溃或无法接收Handler。至于为什么会这样，我们在源码里面找原因。

## 源码解析 ##
1. Handler创建
我们先看Handler的构造方法：
    ```
public Handler(Callback callback, boolean async) {
    //....省略部分代码
    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
    ```
 我们可以看到，Handler的构造方法当中是去获取了一个叫做`Looper`的类对象，如果该对象为空，就会抛出刚才我们上面发生的异常。所以我们需要在创建Handler之前，一定要先执行`Looper.prepare()`。
 那么问题来了，为什么`主线程`就不需要执行`Looper.prepare()`就可以直接创建Handler呢？
 我们可以随意根据代码猜测一下：
 这里Handler的构造方法的代码已经很明显了，`Looper`类是必要的，那么`主线程`可以成功创建Handler，是不是就代表着主线程的`Looper`不为空呢？
![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/zhenxiang_just_one.jpeg)
 是不是主线程在初始化的时候`Looper`也跟着初始化了呢！？带着看破一切(瞎猜)的思路，我们来看`主线程`的初始化源码。
经过了长时间的Google，我们知道了主线程类叫做:`ActivityThread`。
 该类当中有main方法：
    ```
public static void main(String[] args) {
    //....省略部分代码
    
    
    Looper.prepareMainLooper();
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    if (false) {
        Looper.myLooper().setMessageLogging(new
                LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    // End of event ActivityThreadMain.
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    Looper.loop();
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
    ```
 我们很自然（惊奇）地发现，在主线程创建的过程中，果然（真的）有与`Looper`类相关的内容。
 这里有重点（敲黑板）：之前说的Looper.loop()后的代码不会执行，这里得到了证实：
    ```
Looper.loop();
throw new RuntimeException("Main thread loop unexpectedly exited");
    ```
 下面异常的意思是：主线程的Looper意外退出。
 也就是当Looper.loop()执行失败的意思，但是当Looper.loop()执行成功时，是不会执行下面的代码的！因为**`Looper.loop()`必须放到方法的最后，否则会导致后面的代码无法执行**。
 好的，接着往下看，点进`prepareMainLooper()`会发现，其实内部就是调用了`prepare()`：
    ```
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
    ```
 所以到现在，我们解决了我们的第一个问题：为什么主线程当中不需要执行`Looper.prepare()`。
 接着，我们去浏览`Looper.prepare()`：
    ```
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    //new Looper()并设置给当前线程
    sThreadLocal.set(new Looper(quitAllowed));
}
    ```
 没什么东西，前面对线程当中的Looper进行了判空，如果不为空则会抛出`RuntimeEception`。
 这也就是说，每个线程当中只能有一个Looper，当你尝试去创建第二时，就会发生异常，所以`Looper.prepare()`每个线程中只能调用一次。
 后面则new了Looper并且设置给当前线程。
`new Looper()`中初始化了`MessageQueue`：
    ```
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
    ```
 到这里，Handler的创建就完成了！

2. Handler发送Meesage
Handler的发送方法有很多，查看源码会发现，最后调用的都是同一个方法：`MessageQueue`的`enqueueMessage()`：
    ```
boolean enqueueMessage(Message msg, long when) {
//...省略一些判空操作

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
    ```


>未完