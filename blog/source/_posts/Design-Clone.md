---
title: 程序运行更高效-原型模式
date: 2017-07-15 14:52:23
tags:
   - 编程思想
---
## 模式介绍 ##
**原型模式**是一种创建式模式。
用户可以从一个样板对象中复制出一个内部属性一致的对象，这个过程也就是我们常说的“克隆”。
被复制出的对象就是我们所说的“原型”，这个“原型”是可以进行定制的。
**原型模式**多用于创建复杂、构造耗时的对象，**因为在这种情况下，利用原型模式复制一个已经存在的对象可以使程序运行更高效**。
## 应用场景 ##
- 一个对象初始化时需要消耗非常多的资源；
- 一个对象需要提供给多个对象访问，并且多个对象调用时需要进行定制。

需要注意的是，复制操作并不一定比new快，只有当new对象较为耗时或成本较高时，通过复制才能获得效率上的提升。
因此，在使用**原型模式**时需要对对象的构建成本进行一些效率测试。
## 简单示例 ##
如何实现复制操作？
非常简单，Java提供了一个叫做**Cloneable**的接口，我们只需实现该接口，重写它的`clone()`方法即可。
现在我们来举个简单实例来演示**原型模式**：
假设我们现在有一篇文档对象：
```
public class WordDocument{
    //文本
    private String text;
    //作者
    private String author;
    //图片集合
    private ArrayList<String> imageList;

    //....省略get、set
}
```
里面包含了作者、文本以及图片合集，现在我们要让这个文档实现复制功能：
```
public class WordDocument implements Cloneable {
    //文本
    private String text;
    //作者
    private String author;
    //图片集合
    private ArrayList<String> imageList;
    
    @Override
    protected WordDocument clone() {
        WordDocument document = null;
        try {
            document = (WordDocument) super.clone();
            document.text = this.text;
            document.author = this.author;
            document.imageList = this.imageList;
            return document;
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
    
    //....省略get、set
}
```
非常简单，我们让**WordDocument**实现了**Cloneable**接口并重写了`clone()`方法，在`clone()`中进行对象的克隆操作。
那么接下来我们来实际演示下克隆效果：
```
//创建对象
WordDocument document = new WordDocument();
document.setAuthor("孟远");
document.setText("这是一篇极好的文章。");
document.addImage("图0");
document.addImage("图1");
//打印创建的对象
tv_clone_0.append("创建的对象:\n" + document.toString());
//克隆对象
WordDocument cloneBean = document.clone();
//打印克隆的对象
tv_clone_0.append("克隆的对象:\n" + cloneBean.toString());
//修改克隆的对象
cloneBean.setAuthor("黑色小老虎");
cloneBean.addImage("新加图片2");
//再次打印2个对象
tv_clone_0.append("修改克隆对象:\n" + cloneBean.toString());
tv_clone_0.append("最开始的对象:\n" + document.toString());
```
这段代码我们创建了一篇文章，之后拷贝了这篇文章并修改了拷贝文章。
在这期间，一共进行了4次对象的`toString`：
1. 创建对象完成时打印了创建对象；
2. 拷贝完成时打印了拷贝对象；
3. 修改拷贝对象完成时打印了拷贝对象；
4. 最后再次打印最初创建的对象。

这里请注意，我们修改拷贝对象时，修改了作者姓名并且添加了一张图片。
最后打印的结果如下：
![](http://7xvzby.com1.z0.glb.clouddn.com/design_clone_0.png)
细心的同学会发现，我们修改了拷贝对象的作者昵称，原对象没有受到影响。但是我们增加一张图片到拷贝对象的集合中时，原对象也发生了变化。
这就牵扯到了**浅拷贝**和**深拷贝**。

## 深拷贝 ##
上述简单实例，是使用了**浅拷贝**来实现的。所谓**浅拷贝**就是直接引用原对象中的嵌套对象，不会去进行创建。
大家应该都知道**对象引用**的问题：
```
Bean a = new Bean("孟远","23","男");
Bean b = a;
```
此时b对象引用了a对象，也就是说其实a和b两个对象在堆内存中指向的是同一个地址，当修改b时，a也必定跟着发生变化。
同理我们再回头重新看下**WordDocument**的`clone()`代码：
```
 @Override
 protected WordDocument clone() {
     WordDocument document = null;
     try {
         document = (WordDocument) super.clone();
         document.text = this.text;
         document.author = this.author;
         //问题所在，直接引用当前对象的List
         document.imageList = this.imageList;
         return document;
     } catch (CloneNotSupportedException e) {
         e.printStackTrace();
     }
     return null;
 }
```
可以发现我们直接引用了当前对象的List，导致在内存中拷贝对象和最初对象的List指向了同一个地址。
上面代码就是我们口中的**浅拷贝**。
那么如何解决这个问题？
很简单，使用**深拷贝**，**即内部对象也使用`clone()`：**
```
@Override
protected WordDocument clone() {
    WordDocument document = null;
    try {
        document = (WordDocument) super.clone();
        document.text = this.text;
        document.author = this.author;
        //关建行
        document.imageList = (ArrayList<String>) imageList.clone();
        return document;
    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }
    return null;
}
```
我们点进ArrayList的源码可以发现，ArrayList已经实现了**Cloneable**接口，所以我们直接调用ArrayList的`clone()`即可。
修改完这一行代码之后，再次执行上面演示代码：
![](http://7xvzby.com1.z0.glb.clouddn.com/design_clone_1.png)
完美，可以发现在修改完克隆对象之后，最开始的对象已经不会受到影响。
## 总结 ##
[上述演示代码已经上传至GitHub。](https://github.com/z593492734/Design-Pattern)
**原型模式**是非常简单的一个模式，它的核心问题就是对原始对象进行拷贝，在这个模式的使用过程中需要注意一点就是：深、浅拷贝的问题。
在实际开发过程中，为了减少错误，建议各位读者在使用**原型模式**时尽量使用深拷贝，避免操作副本时影响到原始对象。

## 感谢 ##
《Android源码设计模式解析与实战》 何红辉、关爱民 著