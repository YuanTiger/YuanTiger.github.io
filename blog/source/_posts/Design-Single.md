---
title: 应用最广-单例模式
date: 2017-05-27 10:45:14
tags:
   - 编程思想
---
## 模式介绍 ##
单例模式是应用最广泛的模式之一。
单例模式是为了确保一个类在整个项目中只有一个实例对象。
**单例模式最大的优势就是可以避免资源的浪费**。
比如访问IO和数据库等资源时就应考虑使用单例模式。
## 模式特点 ##
1. 构造方法私有化，使用`private`来修饰；
2. 确保对象有且只有一个，尤其是在多线程的环境下；
3. 通过静态方法或枚举返回已经实例化好的对象。

## 模式示例 ##
实现单例模式的方式有很多，不过核心是不变的，都要严格遵循单例模式的特点。
下面我们来介绍实现单例的方式：
1. 饿汉式

    ```
public class 饿汉式 {
    //自行实例化对象
    private static final 饿汉式 ourInstance = new 饿汉式();
    //通过静态方法返回对象
    public static 饿汉式 getInstance() {
        return ourInstance;
    }
    //构造方法私有化，不能通过new来创建对象
    private 饿汉式() {
    }
}
    ```
 值得一提的是，AndroidStudio在创建类时指定该类为单例的时候，默认就是使用饿汉式：
![](http://7xvzby.com1.z0.glb.clouddn.com/design_single_01.png)
 饿汉式写起来非常简单、快捷，但是缺点也显而易见：
在类初始化的时候，对象就已经创建好了。
如果说我们没有用到该类，就会造成资源的浪费。

2. 懒汉式
    ```
public class 最简单的懒汉式 {
    //全项目唯一的对象
    private static 最简单的懒汉式 ourInstance;

    //构造方法私有化
    private 最简单的懒汉式() {

    }

    //通过静态方法来返回对象
    public static 最简单的懒汉式 getInstance() {
        //在调用该方法时进行判空，在对象为null时创建对象
        if (ourInstance == null) {
            ourInstance = new 最简单的懒汉式();
        }
        return ourInstance;
    }
}
    ```
 这就是单例中懒汉式的最基本写法。
 比起饿汉式，最大的优势就是不会造成资源的浪费。因为只有在用到时，才会进行对象的实例化。
 但是就上面的写法而言，还存在一个很致命的问题：
 **在多线程同时调用时，会出现多个实例对象的情况**。
[Demo](https://github.com/YuanTiger/Design-Pattern)里有对应的测试代码，出现的概率很小，但是确实会出现。
解决这个问题的方式也很简单，为静态方法添加同步锁：
    ```
//通过静态方法来返回对象
public static synchronized 同步锁的懒汉式 getInstance() {
     //在调用该方法时进行判空，在对象为null时创建对象
        if (ourInstance == null) {
            ourInstance = new 同步锁的懒汉式();
        }
         return ourInstance;
}
    ```
 `synchronized`就是同步锁的关键字，加上该关键字，代表着该方法同时只能在唯一的一个线程中运行。
比如当10个线程去调用`同步锁的懒汉式.getInstance()`时，只有当第1个线程完成访问时，第2个线程才会开始执行该方法。当第1个线程访问完成后，单例对象就已经创建完成，所以第2个线程就会直接返回该对象，不会再去创建，这就保证了线程安全。
这样确实解决了我们所说的线程安全的问题，但是这种做法明显是低效率的：
我们的目的是保证项目中有且只有一个对象，上述代码确实实现了这个目的。但是当对象创建成功后，我们希望多线程访问的时候应该是异步高效、同时执行的的，而不是像上面那样队列式的，我要等你用完我才能用。所以就有了**双重校验锁**的懒汉式：
    ```
public static 同步锁的懒汉式 getInstance() {
    if (ourInstance == null) {
        synchronized (new Object()) {
            if (ourInstance == null) {
                ourInstance = new 同步锁的懒汉式()
            }
        }
    }
    return ourInstance;
}
    ```
 这种写法可以完美解决多线程效率低下的问题，那么到底是如何解决的？
 **双重校验锁**指的是会进行两次判空操作：
	```
ourInstance == null
	```
 一次在同步锁外，一次在同步锁内。
 有的看官就有疑问了：两次判空？
 首先是`synchronized`关键字，我们删除了方法的同步锁，将其移动到了方法内部，对
 	```
ourInstance = new 同步锁的懒汉式()
	```
 单独加锁。这就代表着我们这个方法本身已经不是线程安全了，会有多个线程同时访问外层的**if**。如果同步锁内部没有判空，就会有多个线程等待对象创建，就会生成多个实例对象。
所以**双重校验锁**的每一步都非常关键，必不可少。
**双重校验锁**的写法主要是为了在多线程创建对象时，用同步锁来保证对象的唯一。当对象创建完成后，同步锁外层的判空操作就不成立了，那么会直接返回对象，整个方法就与同步锁无关，多线程访问时也就不需要等待了。
**双重校验锁**懒汉式，看起来已经非常完美了！
但是，很遗憾。
因为JVM存在**指令重排**的优化，又会产生新的问题。
 ![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/heirenwenhao_2.jpg)
 **指令重排**是JVM为了提高程序运行效率。
JVM规范规定，**指令重排序可以在不影响单线程程序执行结果的情况下改变代码执行顺序**。
该处会产生**指令重排**的代码是
	```
ourInstance = new 同步锁的懒汉式();
	```
 这句代码在JVM看来，主要是做了以下三件事情：
  （1）给`ourInstance`分配内存；
  （2）调用构造方法创建对象，对对象进行初始化；
  （3）将`ourInstance`对象指向JVM分配的内存空间（此步完成之后，`ourInstance`就是非null了）。
因为JVM存在**指令重排**，所以在不影响最终结果的情况下，JVM会选择性能最优的的顺序执行：
也就是说，上面三件事情，执行的顺序可能是1-2-3，也有可能是1-3-2。
1-2-3，1-3-2，有区别吗？
在结果上来看，没有任何区别。
但是在多线程的情况下，是有风险的：
假设线程x的执行顺序是1-3-2，当3执行完成时，`ourInstance`就已经不为空了，但是2还没有执行完成时，线程y介入了。此时线程y会发现`ourInstance`已经不为null了，但是其实`ourInstance`的初始化工作并未完成，这样很明显就会产生异常。
解决方法也非常简单，利用`volatile`关键字即可：
    ```
public class 完美的懒汉式 {
    //全项目唯一的对象
    //volatile关键字，禁止指令重排
    private volatile  static 完美的懒汉式 ourInstance;

    //构造方法私有化
    private 完美的懒汉式() {

    }

    //通过静态方法来返回对象
    public static 完美的懒汉式 getInstance() {
        //在调用该方法时进行判空，在对象为null时创建对象
        if (ourInstance == null) {
            synchronized (new Object()) {
                if (ourInstance == null) {
                    ourInstance = new 完美的懒汉式();
                }
            }
        }
        return ourInstance;
    }
    ```
 上述代码就是一个完美的懒汉式了，利用`volatile`关键字来禁止JVM的**指令重排**。
 
3. 枚举（Enum）
    ```
 public enum 枚举单例 {

    INSTANCE;
    
    public String getUrl(){
        return "http://www.baidu.com";
    }
}
    ```
 使用起来也非常简单：
    ```
    String url = 枚举单例.INSTANCE.getUrl();
    ```
 简直完美啊！简单易用，代码清晰！
但是，很少有人选择用枚举单例。
可能。。。。。？
 
## 总结 ##
简单回顾一下：
单例模式是保证了一个类在一个项目中有且只有一个实例对象。
这样做的目的是为了节省内存的开支。
单例模式的写法主要有：
- **项目初始化时就创建好的饿汉式**
- **在第一次使用时才进行创建、但要注意线程安全的懒汉式**
- **使用非常简单的枚举**
##
## 感谢 ##
[Jark's Blog-如何正确地写出单例模式
](http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/)
《Android源码设计模式解析与实战》 何红辉、关爱民 著