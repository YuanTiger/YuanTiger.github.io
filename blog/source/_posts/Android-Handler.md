---
title: Android_Handler机制
date: 2017-05-15 14:57:07
tags:
   - Android
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
在Meesage入队的时候，也就是执行`MessageQueue.enqueueMessage()`方法时，`enqueueMessage()`有一个`nativeWeak()`的native方法，如果有消息进入，并且Looper是休眠状态，则会执行该方法唤醒Looper:
```
// We can assume mPtr != 0 because mQuitting is false.
if (needWake) {
    nativeWake(mPtr);
}
```
5. 整体流程
先调用`Looper.prepare()`创建Looper，在创建的同时会自动调用`Looper.loop()`执行死循环loop()。注意`Looper.loop()`一定放到代码的最后一行。
死循环中会执行`MessageQueue.next()`方法去取出队列中的消息，当消息为空时，`MessageQueue.next()`方法中会执行`nativePollOnce()`的native方法休眠`Looper.loop()`死循环。当有新的消息插入到`MessageQueue`中，也就是调用`MessageQueue.enqueueMessage()`方法，这个方法当中会判断Looper是否是休眠状态，如果是休眠状态会执行`nativeWeak()`的native方法来唤醒Looper()。

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
Handler的发送方法有很多，包括发送延时Handler、及时Handler、空Hanlder等等。
查看源码会发现，所有发送方法最后调用的都是同一个方法：`MessageQueue`的`enqueueMessage()`。
有的看官就会问：Handler中怎么会有`MeesageQueue`？
这个在上面`new Looper()`的源码中已经体现了：
`new Handler()`中`new Looper()`，
`new Looper()`中`new MessageQueue()`。
所以其实初始化Handler的同时，`Looper`和`MeesageQueue`都已经初始化完成了。
下面我们来看消息入队方法`MessageQueue.enqueueMessage()`的全部源码：
```
boolean enqueueMessage(Message msg, long when) {
        //Meesage是否可用
        //这里的msg.target指的就是发送该Message的Handler
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }
        //同步锁
        synchronized (this) {
            //判断是否调用了quit()方法，即取消信息
            //如果调用了，则其实Handler的Looper已经销毁，无法发送消息
            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                msg.recycle();
                return false;
            }

            //将消息添加到MessageQueue的具体操作
            //每来一个新的消息，就会按照延迟时间的先后重新进行排序
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

            //如果Looper.loop()是休眠状态
            //则调用native方法唤醒loop()
            //---重点---Looper的唤醒
            if (needWake) {
                nativeWake(mPtr);
            }
        }
        return true;
    }
```
 将Message入队到`MeesageQueue`的核心代码，就是这些。
 根据注释也基本能理解该方法的作用。
3. 取出Message
取出Meesage想必大家都知道在哪里取出：`Looper.loop()`：
```
public static void loop() {
    //Looper的判空
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;
    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();
    //取出Message，死循环
    for (;;) {
        //取出Meesage的核心代码
        //在当next()返回为空时，next()中会休眠loop()
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }
        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }
        msg.recycleUnchecked();
    }
}
```
 `Looper.loop()`的核心就是取出Message，而取出Message的核心就是`MeesageQueue.next()`：
```
 Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
        //唤醒Looper.loop()的native方法
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the 
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it 
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.M
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }
            // Process the quit message now that all pending messages have been hand
            if (mQuitting) {
                dispose();
                return null;
            }
            // If first time idle, then get the number of idlers to run.
            // Idle handles only run if the queue is empty or if the first message
            // in the queue (possibly a barrier) is due to be handled in the future.
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                mBlocked = true;
                continue;
            }
            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCo
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }
        // Run the idle handlers.
        // We only ever reach this code block during the first iteration.
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler
            boolean keep = false;
            try {
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf(TAG, "IdleHandler threw exception", t);
            }
            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }
        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;
        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
```
 取出Meesage的代码有些多，大部分都是一些优化逻辑：next() 方法还做了其他一些事情，这些其它事情是为了提高系统效果，利用消息队列在空闲时通过 idle handler 做一些事情，比如 gc 等等。

## 结语 ##
到这里Handler源码的浅析就结束了，总结在最上方，建议各位看官再去看一下总结加深印象。

