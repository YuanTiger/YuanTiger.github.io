---
title: 创建型设计模式-抽象工厂模式
date: 2017-08-02 13:52:48
tags:
   - 编程思想
---
## 模式介绍 ##
前段时间介绍了[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)。
本次介绍的模式非常容易和[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)混淆，并且复杂度也有一定的提升，叫做**抽象工厂模式**。
其实在现实生活中，工厂都会生产某一样具体的产品，不存在说抽象的工厂。
那么该模式的含义到底是什么呢？
**抽象工厂模式**的出现最早是为了解决不同操作系统下的图形化处理，这就好比iOS中的Button、Android中的Button、WindowPhone中的Button一样，虽然都是Button，但系统的不同，会导致Button也存在差异。
这就引出了**抽象工厂模式**的定义：
为创建出**一组相互依赖**的对象提供一个封装接口。
定义中有几个关键字，需要注意：
- 一组：这是和[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)的重要区别所在，**抽象工厂模式**用于生产多个产品，而不是一个。如果生产一个产品，请使用[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)或[建造者模式](http://www.jianshu.com/p/07bed15cd13c)。
- 相互依赖：生产出一组产品之后，这组产品**一定**是相互依赖的，注意是**一定**。就好比上面的例子中，Android系统能使用iOS系统的Button吗？答案是否定的，因为iOS的Button依赖于iOS系统。

解释完名词，想必大家对**抽象工厂模式**的使用场景就有了一些认知：
当产品很多，并且有特定的关联可以进行抽象，就可以使用**抽象工厂模式**。



## 模式构成 ##
**抽象工厂模式**的主要角色还是4个：
- AbstractFactory：抽象工厂，包含一组产品的生产抽象，**每一个方法都应对应一个产品**。
- ConcreteFactory：抽象工厂实现，应包含生产一组产品的具体实现。
- AbstractProduct：一组产品中的一个产品抽象。
- ConcreteProduct：具体的产品实现。

## 模式示例 ##
现在我们开始实现在上述介绍中提到的例子。
我们先来将思路捋一捋：
![对应关系](http://7xvzby.com1.z0.glb.clouddn.com/DesignModel/design_abs_factory_0.png)

1. 我们产品包含：操作系统、Button。
2. 工厂应该生产一组产品，即生产操作系统和Button。
3. 生产出来的一组产品应相互依赖，并且和另一组产品相互独立。

接下来，我们就按照思路来实现**抽象工厂模式**。
首先我们来初始化抽象产品**AbstractProduct**：
操作系统System：
```
public interface SystemAbsProduct {

    //获取系统型号
    String getSystem();

}
```
Button：
```
public interface ButtonAbsProduct {

    //按钮响应
    String getButtonName(SystemAbsProduct system);

}
```
我们可以发现，Button与操作系统存在依赖关系，这种依赖关系的体现由开发人员自己来控制，我这里只是个简单实例。
两个产品的抽象已经有了，我们先不急着去实现它们。
接下来我们来定义抽象工厂AbstractFactory，它应该有两个方法，分别返回系统和Button两个产品：
```
public interface AbsFactory {

    //创建操作系统
    SystemAbsProduct createSystem();

    //创建Button
    ButtonAbsProduct createButton();
}
```
到这里应该都没有什么问题，抽象工厂、抽象产品都已经创建完成了。
接下来我们想要分别构建出WindowPhone系统、Android系统、iOS系统下的Button。
我们应该能想到，必须要先要有具体的产品，才能用具体工厂来生产。
所以接下来，我们要先创建具体的产品：
操作系统
```
public interface SystemAbsProduct {

    //获取系统型号
    String getSystem();

    public class WindowPhone implements SystemAbsProduct{

        @Override
        public String getSystem() {
            return App.context.getString(R.string.window_phone);
        }
    }
    public class iOS implements SystemAbsProduct{

        @Override
        public String getSystem() {
            return App.context.getString(R.string.ios);
        }
    }

    public class Android implements SystemAbsProduct{

        @Override
        public String getSystem() {
            return App.context.getString(R.string.android);
        }
    }
}
```
Button：
```
public interface ButtonAbsProduct {

    //按钮响应
    String getButtonName(SystemAbsProduct system);
    
    public class WindowPhoneButton implements ButtonAbsProduct {

        @Override
        public String getButtonName(SystemAbsProduct system) {
            if (system.getSystem().equals(App.context.getString(R.string.window_phone))) {
                return "点击了WindowPhone系统的Button";
            }
            return App.context.getString(R.string.system_error);
        }
    }

    public class IOSButton implements ButtonAbsProduct {

        @Override
        public String getButtonName(SystemAbsProduct system) {
            if (system.getSystem().equals(App.context.getString(R.string.ios))) {
                return "点击了iOS系统的Button";
            }
            return App.context.getString(R.string.system_error);
        }
    }

    public class AndroidButton implements ButtonAbsProduct {

        @Override
        public String getButtonName(SystemAbsProduct system) {
            if (system.getSystem().equals(App.context.getString(R.string.android))) {
                return "点击了Android系统的Button";
            }
            return App.context.getString(R.string.system_error);
        }
    }
}
```
因为类的数量庞大，所以我选择使用内部类的形式来实现ConcreteProduct角色。
我们可以发现，在Button角色中，我们进行了简单的判断，当操作系统不匹配时，会提示用户，这就类似于运行崩溃的效果。
接下来，我们来创建最后一个角色：ConcreteFactory。
它应该有多个，每一个负责创建一组具体的产品，也就是说有多少组产品，就应该有多少个具体工厂：
WindowPhoneFactory：
```
public class WindowPhoneFactory implements AbsFactory {
    @Override
    public SystemAbsProduct createSystem() {
        return new SystemAbsProduct.WindowPhone();
    }

    @Override
    public ButtonAbsProduct createButton() {
        return new ButtonAbsProduct.WindowPhoneButton();
    }
}
```
IOSFactory：
```
public class IOSFactory implements AbsFactory {
    @Override
    public SystemAbsProduct createSystem() {
        return new SystemAbsProduct.IOS();
    }

    @Override
    public ButtonAbsProduct createButton() {
        return new ButtonAbsProduct.IOSButton();
    }
}
```
AndroidFactory：
```
public class AndroidFactory implements AbsFactory {
    @Override
    public SystemAbsProduct createSystem() {
        return new SystemAbsProduct.Android();
    }

    @Override
    public ButtonAbsProduct createButton() {
        return new ButtonAbsProduct.AndroidButton();
    }
}
```
三个具体工厂分别具体产生三组相互依赖的产品。
至此，简单的**抽象工厂模式**已经构建完成。
但是最关键的还是使用，接下来我们来写一些简单的测试代码：
```
//构建WindowPhone具体工厂
AbsFactory iOSFactory = new IOSFactory();
//构建操作系统
SystemAbsProduct iOSSystem = iOSFactory.createSystem();
//构建按钮
ButtonAbsProduct iOSButton = iOSFactory.createButton();
//调用依赖
Toast.makeText(this, iOSButton.getButtonName(iOSSystem), Toast.LENGTH_SHORT).show();
```
创建iOS系统工厂，接着创建iOS系统和Button，最后调用Button的点击，会发现成功提示出调用成功。
如果我们使用错误的依赖：
```
AbsFactory androidFactory1 = new AndroidFactory();
SystemAbsProduct androidSystem1 = androidFactory1.createSystem();
ButtonAbsProduct androidButton1 = androidFactory1.createButton();
AbsFactory iOSFactory1 = new IOSFactory();
SystemAbsProduct iOSSystem1 = iOSFactory1.createSystem();
ButtonAbsProduct iOSButton1 = iOSFactory1.createButton();
//使用Android的Button时，传入iOS操作系统
Toast.makeText(this, androidButton1.getButtonName(iOSSystem1), Toast.LENGTH_SHORT).show();
```
上述演示代码，会提示系统不匹配。

## 总结 ##
**抽象工厂模式**的最大优势就是分离了接口与实现，客户端使用工厂来创建所需对象，实现面向产品的接口编程，使其在切换产品类时更加灵活、容易。
当然缺点也非常明显，一是类文件的增加，二是不太容易扩展产品类，因为每添加一个新产品，都要修改工厂，随之所有工厂实现都要修改。
**抽象工厂模式**与[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)的区别也很明显：
**抽象工厂模式**用来生产一组相互依赖的产品。
[工厂方法模式](http://www.jianshu.com/p/cf3b3335af4b)用来批量生产一种产品。
[抽象工厂模式的相关代码已经上传至GitHub](https://github.com/z593492734/Design-Pattern)，需要的同学可以下载参考。

## 感谢 ##

[抽象工厂模式和工厂模式的区别-caoglish的回答](https://www.zhihu.com/question/20367734/answer/82361745?from=singlemessage&isappinstalled=0)

《Android源码设计模式解析与实战》 何红辉、关爱民 著
