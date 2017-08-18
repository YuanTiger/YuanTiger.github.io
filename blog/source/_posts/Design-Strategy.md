---
title: 实现动态替换-策略模式
date: 2017-08-02 17:15:00
tags:
   - 编程思想
---
## 模式介绍 ##
通常，我们实现一个功能，可以有多种策略。
我们可以根据实际需求来选择最合适的策略。
针对这种情况，一种常规的方法就是将多种策略封装在一个类中，每个策略对应一个方法，在使用时通过if-else进行情况判断，不同情况调用不同方法来实现不同的策略。
这种实现方法我们可以称之为硬编码。

然而，当策略越来越多的时候，这个类就会变得臃肿，并且由于if-else等复杂逻辑的存在，在维护时会更容易产生错误，维护成本就会变高。
并且这种写法明显违反了[面向对象](http://www.jianshu.com/p/2c5826aa996c)中的开闭原则。
如果我们将这些策略的共同点抽象出来，提供一个统一的接口，不同的策略有着不同的实现，这样在客户端就可以通过注入不同的实现对象来实现动态替换。
这种模式的可扩展性、可维护性都是极高的。这，就是我们要说的**策略模式**。
总感觉，这模式，不就是多态嘛。。

## 模式定义 ##
针对一个功能，将每一种解决方案封装起来，而且使它们可以相互替换，这就是**策略模式**。

## 使用场景 ##
- 针对同一类型问题的多种处理方式，仅仅是具体行为存在差异；
- 需要安全地封装多种同一类型的操作时；
- 同一抽象类有多个子类，同时又需要使用if-else或swithc-case来选择具体子类时。

## 模式角色 ##
- Strategy：策略抽象。
- ConcreteStrategy：具体策略，针对一个问题，有多少种策略，就应有多少个具体策略。

## 模式示例 ##
2014年12月28日北京提高了公交、地铁的价格，不再是单一票价，而是分距离计价。
我们就根据上述需求，来做一个出行价格计算器。
话不多说，我们来实现这个功能：
![测试UI](http://7xvzby.com1.z0.glb.clouddn.com/DesignModel/design_strategy_ui.png)
简单做了计算器的UI，它应该提供具体出行方式的选择，很明显，我们提供了三种。
并且有一个出行距离的输入框。
在获取到出行方式和出行距离后，我们就可以来计算具体价格。
需求很明确，接下来我们要做的就是先实现各种出行方式下的价格计算：
```
public class PriceCaculateController {

    /**
     * 根据出行方式，选择距离的计算方法
     */
    public static float cacluatePrice(int km, int typeMode) {
        switch (typeMode) {
            case Constant.TYPE_BUS:
                return caculateBusPrice(km);
            case Constant.TYPE_SUBWAY:
                return caculateSubwayPrice(km);
            case Constant.TYPE_TAX:
                return caculateTaxPrice(km);
        }
        return 0;
    }

    /**
     * 计算出租车价格
     * 小于3Km，定价9元，大于3km，每1km + 1元
     *
     */
    private static float caculateTaxPrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 3) {
            return 9;
        }
        return 9 + (km - 3);
    }

    /**
     * 计算公交车价格
     * 小于5Km，定价2元，小于10km，定价3元，其余4元
     */
    private static float caculateBusPrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 5) {
            return 2;
        }
        if (km <= 10) {
            return 3;
        }
        return 4;
    }
    /**
     * 计算地铁价格
     * 小于5Km，定价3元，小于10km，定价4元，小于15Km，定价5元，其余6元
     */
    private static float caculateSubwayPrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 5) {
            return 3;
        }
        if (km <= 10) {
            return 4;
        }
        if (km <= 15) {
            return 5;
        }
        return 6;
    }
}
```
`PriceCaculateController`中包含了所有出行方式的计算细节。
细心的同学可以发现，我将`PriceCaculateController`中的所有计算方法私有化，向外暴露了一个`cacluatePrice(int km, int typeMode)`方法。
用户在使用时，仅需告诉我们出行距离与出行方式，我们就可以完全自动计算出价格，下面是使用的代码：
```
float price = PriceCaculateController.cacluatePrice(20,1);
```
这样我们就实现了页面与计算功能的完美解耦，页面完全不关心计算的细节。
接下来我们在页面上添加基础判断之后，调用`PriceCaculateController.cacluatePrice `即可。
我们可以来看看具体效果：
![具体效果](http://7xvzby.com1.z0.glb.clouddn.com/DesignModel/design_strategy_gif_ui.gif)
至此，我们的价格计算器已经开发完成了。
很明显，我们的核心代码全在`PriceCaculateController`中。
随着出行方式的增加、价格计算的优化，`PriceCaculateController`必定会变得臃肿不堪。
为了将`PriceCaculateController`功能进行拆分，我们就要使用今天讲到的**策略模式**。
让我们来回顾**策略模式**中的角色：
**一个策略抽象以及多个具体策略**。

接下来我们就将价格计算功能进行抽象：
```
public interface StrategyCaculate {

    /**
     * 根据距离计算价格
     */
    float caculatePrice(int km);
}
```
策略抽象非常简单。
接下里我们来实现具体策略，有多少种出行方式，就应该有多少种具体策略：
公共汽车(Bus):
```
public class BusStrategyCaculate implements StrategyCaculate {
    @Override
    public float caculatePrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 5) {
            return 2;
        }
        if (km <= 10) {
            return 3;
        }
        return 4;
    }
}
```
出租车(Tax):
```
public class TaxStrategyCaculate implements StrategyCaculate {
    @Override
    public float caculatePrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 3) {
            return 9;
        }
        return 9 + (km - 3);
    }
}
```
地铁(Subway):
```
public class SubwayStrategyCaculate implements StrategyCaculate {
    @Override
    public float caculatePrice(int km) {
        if (km <= 0) {
            return 0;
        }
        if (km <= 5) {
            return 3;
        }
        if (km <= 10) {
            return 4;
        }
        if (km <= 15) {
            return 5;
        }
        return 6;
    }
}
```
三种出行方式对应三个具体策略。
当有新的出行方式时，我们只需创建新的类去实现策略抽象即可。
简单写一些测试代码：
```
StrategyCaculate caculate = null;
switch (typeMode) {
    case Constant.TYPE_BUS:
        caculate = new BusStrategyCaculate();
        break;
    case Constant.TYPE_SUBWAY:
        caculate = new SubwayStrategyCaculate();
        break;
    case Constant.TYPE_TAX:
        caculate = new TaxStrategyCaculate();
        break;
}
assert caculate != null;
return caculate.caculatePrice(km);
```
可以发现，这里我们又运用多态的特性，根据不同的出行方式，创建不同的子类，接着调用子类中的价格计算。
最后的效果和之前的效果是一致的。
[相关代码已经提交至GitHub](https://github.com/YuanTiger/Design-Pattern)。
## 总结 ##
有些人会有疑惑了，我们最开始的`PriceCaculateController`并不复杂，并且一个方法对应一种出行方式，清晰明了，为什么非要改成这个样子?
对于这个问题，我们现在的出行方式仅仅有三种、并且价格计算的逻辑非常简单。
如果现在继续添加出行方式：飞机、自驾、高铁、大巴等。
如果优化出租车的价格计算逻辑：出租车3Km以内9元，之后每1Km加1元，燃气费2元，堵车服务费每分钟0.1元，夜晚11点之后价格提升，每个城市的价格不同等等。
如果将上述逻辑全部写出，那么就出租车价格计算的逻辑，就要有上百行的代码。
如果你们公司是一家专业的出行价格计算公司，这些计算细节、出行方式都必定要涵盖到。
那么将来，`PriceCaculateController`的代码量，会有多大？
如果我们运用了**策略模式**，项目的目录结构就变成了：
![策略模式目录结构](http://7xvzby.com1.z0.glb.clouddn.com/DesignModel/design_strategy_rule.png)
当某种交通工具的价格计算发生问题，我们仅仅去找对应的具体策略即可。并且避免产生了`PriceCaculateController`这种代码量庞大的类。
**策略模式**很好地遵循了开闭原则，注入不同的具体策略，会有不同的效果，从而达到很好的扩展性。
使用**策略模式**的优点有很多：
- 结构清晰明了，使用简单；
- 降低耦合度，扩展方便；
- 封装彻底，数据更为安全。

所以，在你能预见到某个功能会有扩展的需求时，并且使用场景符合**策略模式**的使用场景，还是强烈建议你使用**策略模式**的。

## 感谢 ##

《Android源码设计模式解析与实战》 何红辉、关爱民 著