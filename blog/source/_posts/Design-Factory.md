---
title: 应用最广-工厂方法模式
date: 2017-07-24 13:57:55
tags:
   - 编程思想
---
## 模式介绍 ##
**工厂方法模式**是应用最广泛的模式之一，也是创建型模式之一。
**工厂方法模式**指的是定义出一个用于创建对象的接口，让子类决定实例化哪个类。听起来可能不太懂，没关系，往下看慢慢就明白了。
**工厂方法模式**是一种结构简单的模式，在我们平时开发中应用非常广泛，也许你并不知道，但你已经使用了无数次该模式。

## 使用场景 ##
在任何需要生成复杂对象的地方，都可以用使用**工厂方法模式**。简单对象无需使用**工厂方法模式**，更无需使用[**建造者模式**](http://www.jianshu.com/p/07bed15cd13c)，因为这样反而会损失性能。

## 模式构成 ##
**工厂方法模式**的成员构成如下：
- Product：商品功能的抽象。
- ConcreteProduct：商品功能的具体实现。
- Factory：生产商品的工厂抽象。
- ConcreteFactory：生产商品的工厂具体实现。

需要注意的是，**工厂方法模式**中的Product和[**建造者模式**](http://www.jianshu.com/p/07bed15cd13c)中的Product是有区别的：
前者的Product指的是商品的**功能**，后者指的是商品的**属性**。
也就是两者Product中封装的内容是不一致的，是两个完全不同的模式结构。

## 模式示例 ##
如果将汽车的功能抽象出来，其实都是一致的，无非是有些高级功能，有的汽车有，有的汽车没有。
但是最基本的功能，比如驾驶、大灯、雨刷等功能，是所有汽车都应该具备的。
不过就算是这些都具备的功能，也会因车的品牌、型号的不同，存在差异。
这些就是后话了，我们现在就开始用**工厂方法模式**的代码来实现上述示例。
先来创建汽车Product，它应该包含汽车所有的功能：
```
public interface CarProduct {
    /**
     * 汽车-通用功能封装
     * 开始驾驶
     */
    void drive();
    /**
     * 汽车-通用功能封装
     * 打开车的大灯
     */
    void openHeadlamps();
    
    //....省略其他功能

}
```
可以看出，我们的汽车商品包含驾驶以及大灯两个功能。
两个最简单的功能，在不同品牌、不同型号的车下都会存在差异：
奥迪A6L：
```
public class AudiA6L implements CarProduct{
    @Override
    public void drive() {
        Log.i("factory","奥迪A6L，平稳起步。");
    }

    @Override
    public void openHeadlamps() {
        Log.i("factory","奥迪A6L，尊享华丽大灯打开！");
    }
}
```
奔驰E260L：
```
public class BenzE260L implements CarProduct {
    @Override
    public void drive() {
        Log.i("factory","奔驰E260L，弹射起步！");
    }

    @Override
    public void openHeadlamps() {
        Log.i("factory","奔驰E260L，标配大灯打开。");
    }
}
```
我们创建了两款汽车，分别去实现了`CarProduct`，并且实现了各自功能的细节。
栗子而已，各位看官可不要因为奔驰E260L能不能弹射起步和我撕啊。
![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/%E9%80%80%E5%87%BA%E8%A3%85%E9%80%BC%E7%95%8C.jpg)
如果还有其他类型的汽车，我们只需要接着创建类去实现`CarProduct`即可。
接下来我们就要创建汽车的生产工厂了：
```
public abstract class CarFactory {

    /**
     * 创建CarProduct
     *
     * @return CarProduct
     */
    public abstract CarProduct createProduct();
}
```
可以看出我们的抽象工厂中，返回了**抽象**的商品对象。
那么我们应该在工厂实现中，实现创建商品的具体细节：
奥迪A6L的工厂：
```
public class AudiA6LFactory extends CarFactory {
    @Override
    public CarProduct createProduct() {
        return new AudiA6L();
    }
}
```
奔驰E260L的工厂：
```
public class BenzE260LFactory extends CarFactory {
    @Override
    public CarProduct createProduct() {
        return new BenzE260L();
    }
}
```
因为示例比较简单，仅仅是new的操作，实际创建汽车会更复杂一些。
接下来我们写一些测试代码：
```
//创建奔驰工厂
CarFactory carFactory1 = new BenzE260LFactory();
BenzE260L product1 = (BenzE260L) carFactory1.createProduct();
product1.drive();
product1.openHeadlamps();
//创建奥迪工厂
CarFactory carFactory2 = new AudiA6LFactory();
AudiA6L product2 = (AudiA6L) carFactory2.createProduct();
product2.drive();
product2.openHeadlamps();
```
这里注意一点，创建工厂的代码是利用面向对象思想中**多态思想（继承、重写、父类引用指向子类对象）**来书写的。
接下来打印的Log如下：
```
factory: 奔驰E260L，弹射起步！
factory: 奔驰E260L，标配大灯打开。
factory: 奥迪A6L，平稳起步。
factory: 奥迪A6L，尊享华丽大灯打开！
```
到这里，我们最基本的**工厂方法模式**已经实现了。
但是到这里还有一个缺陷：**商品实现与工厂实现一对一。**
也就是说每多出一个汽车类型、就要为其创建一个具体工厂实现。
为了解决这一问题，我们需要对工厂抽象、工厂细节进行优化，利用**泛型**和**反射**来实现**商品实现与工厂实现多对一：**
```
public abstract class NewCarFactory {
    /**
     * 利用泛型来决定要生成的具体商品
     *
     * @param clz 具体商品的类名class
     * @param <T> 具体商品的类名
     * @return 具体商品类
     */
    public abstract <T extends CarProduct> T createProduct(Class<T> clz);
}
```
上面代码就是经过优化的抽象工厂，创建方法增加了一个参数，决定了要创建的具体商品。
接下来是工厂的具体细节，应该具备创建任意商品的功能：
```
public class CarConcreteFactory extends NewCarFactory {

    @Override
    public <T extends CarProduct> T createProduct(Class<T> clz) {
        CarProduct product = null;
        try {
            product = (CarProduct) Class.forName(clz.getName()).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return (T) product;
    }
}
```
我们用参数得到类名，然后通过反射来创建对应的类。
这样子我们只要有这么一个工厂细节，就能应对所有的商品创建。
写一下测试代码：
```
//创建通用汽车工厂
NewCarFactory carFactory = new CarConcreteFactory();
//创建奔驰E260L
BenzE260L benzE260L = carFactory.createProduct(BenzE260L.class);
//创建奥迪A6L
AudiA6L audiA6L = carFactory.createProduct(AudiA6L.class);
//测试
benzE260L.drive();
audiA6L.drive();
benzE260L.openHeadlamps();
audiA6L.openHeadlamps();
```
只有一个工厂，生产出了不同类型的汽车。
```
factory: 奔驰E260L，弹射起步！
factory: 奥迪A6L，平稳起步。
factory: 奔驰E260L，标配大灯打开。
factory: 奥迪A6L，尊享华丽大灯打开！
```
结果是一致的。
两种实现方式都可以，利用**反射**的方式更简洁而已。
而且上面一对一的写法，还有一个名字：**多工厂方法模式**。

## 总结 ##
**工厂方法模式**对于大家来说是非常好理解的一个模式，即便是第一次听说，只要读者懂点Java知识，理解这个模式绝对不难。
**工厂方法模式**也存在缺陷：每次我们添加新的商品时，都要创建新的类。
如果你辞职了，让不懂**工厂方法模式**的人接手了，这简直是折磨，因为虽然解耦了，但是代码复杂度也随之提升了。
是否使用**工厂方法模式**，需要设计者，你，来决定。
## 感谢 ##

《Android源码设计模式解析与实战》 何红辉、关爱民 著