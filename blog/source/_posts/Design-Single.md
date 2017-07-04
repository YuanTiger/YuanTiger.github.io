---
title: 应用最广的设计模式-单例模式
date: 2017-05-27 10:45:14
tags:
   - 编程思想
---
## 介绍 ##
单例模式是应用最广泛的模式之一。
单例模式是为了确保一个类在整个项目中只有一个实例对象。
**单例模式最大的优势就是可以避免资源的浪费**。
比如访问IO和数据库等资源时就应考虑使用单例模式。
## 特点 ##
1. 构造方法私有化，使用`private`来修饰；
2. 确保对象有且只有一个，尤其是在多线程的环境下；
3. 通过静态方法或枚举返回已经实例化好的对象。

## 示例 ##
实现单例模式的方式有很多，不过核心是不变的，都要严格遵循单例模式的特点。
下面我们来介绍比较流行的实现单例的方式：
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
[Demo]()里有对应的测试代码，出现的概率很小，但是确实会出现。
解决这个问题的方式也很简单，为静态方法添加同步锁：
    ```
//通过静态方法来返回对象
public static 同步锁的懒汉式 getInstance() {
    //在调用该方法时进行判空，在对象为null时创建对象
    if (ourInstance == null) {
        synchronized (new Object()) {
            if (ourInstance == null) {
                ourInstance = new 同步锁的懒汉式();
            }
        }
    }
    return ourInstance;
}
    ```
 
 
 
 
