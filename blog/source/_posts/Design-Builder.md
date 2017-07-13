---
title: 建造者模式（Builder）详解
date: 2017-07-05 15:47:53
tags:
   - 编程思想
---
## 模式介绍 ##
Builder模式是指一步一步来构建出一个复杂的对象。
它允许用户在不知道内部构建细节的情况下，非常精细地控制对象构建流程。
该模式是为了将构建过程非常复杂的对象进行拆分，让它与它的部件解耦，提升代码的可读性以及扩展性。

## 模式结构 ##
**建造者模式**包含如下角色：
- Product：产品角色
- Builder：**抽象**的建造者
- ConcreteBuilder：具体的建造者
- Director：指挥者

## 模式实例 ##
我们来举个生活中的例子来描述**建造者模式**的各个角色：
假设我们现在是一个手机的生产商，各大手机厂商都会来找我们制造手机，比如小米、华为、三星、魅族等。每个厂商的手机配置都不一致，包括CPU型号、内存大小、像素大小等等。
现在我们将这个例子中的对象转化成**建造者模式**的各个角色：
- Product：我们生产的手机就是商品，Product中应该包含手机的各个部件。
- Builder：抽象Builder类，细节全在ConcreteBuilder中。
- ConcreteBuilder：继承了Builder类，这里有手机的组装细节。
- Director：这里的指挥者就是各大手机厂商，我要让你生产一台小米手机并给了你一系列参数，你就要按照参数，生产我想要的手机，其他手机也是同样的同理。

纸上谈兵终觉浅，我们将上述实例转化成代码看一看：
首先是Product，它应该包含手机的零件元素：
```
public class PhoneProduct {

    private String brand;//品牌
    private String CPU;//CPU
    private String memorySize;//内存大小
    private String pixel;//像素大小


    public void setBrand(String brand) {
        this.brand = brand;
    }

    public void setCPU(String CPU) {
        this.CPU = CPU;
    }

    public void setMemorySize(String memorySize) {
        this.memorySize = memorySize;
    }

    public void setPixel(String pixel) {
        this.pixel = pixel;
    }

    @Override
    public String toString() {
        return "品牌：" + brand +
                "\nCPU：" + CPU +
                "\n内存大小：" + memorySize +
                "\n像素大小：" + pixel;
    }

```
我们可以看出，一台手机的构建元素有：品牌、CPU、内存大小以及像素大小，如果还有其他元素，只需要继续添加成员变量即可。
有了手机的构建元素，如何将这些元素拼接成手机，就是Builder要做的事情了：
```
public abstract class PhoneBuilder {
    //构建手机品牌
    public abstract void buildBrand(String brand);
    //构建手机CPU
    public abstract void buildCPU(String cpu);
    //构建手机内存
    public abstract void buildMemorySize(String memorySize);
    //构建手机像素大小
    public abstract void buildPixel(String pixel);
    //将各个零件进行拼接
    public abstract PhoneProduct create();
}
```
可以看到抽象的Builder类中有各个元素的组装方法，并且最后还有一个将手机进行组装的方法：`create()`。
接下来我们使用**ConcreteBuilder**来继承**Builder**并实现具体的组装细节：
```
public class ConcretePhoneBuilder extends PhoneBuilder {
    //商品手机
    private PhoneProduct product = new PhoneProduct();
    
    @Override
    public void buildBrand(String brand) {
        product.setBrand(brand);
    }

    @Override
    public void buildCPU(String cpu) {
        product.setCPU(cpu);
    }

    @Override
    public void buildMemorySize(String memorySize) {
        product.setMemorySize(memorySize);
    }

    @Override
    public void buildPixel(String pixel) {
        product.setPixel(pixel);
    }

    @Override
    public PhoneProduct create() {
        return product;
    }
}

```
**ConcreteBuilder**中包含了每个零件组装和最后拼接过程**create()**的所有细节。
到这里，我们作为一个手机生产商，已经具备了生产手机的能力，接下来就要接待客户了！
平时大家都说，客户是上帝。今天在这里，也是如此。
所以起到主导作用的，就是我们的客户**Director**：
```
public class PhoneDirector {

    private PhoneBuilder builder;

    public PhoneDirector(PhoneBuilder builder) {
        this.builder = builder;
    }

    public PhoneProduct constuct(String brand, String cpu, String memorySize, String pixel) {


        builder.buildBrand(brand);
        builder.buildCPU(cpu);
        builder.buildMemorySize(memorySize);
        builder.buildPixel(pixel);

        return builder.create();
    }
}
```
可以看到**Director**中持有PhoneBuilder对象，这里的**constuct(,,,,)**就相当于客户的需求，告诉我们手机的具体配置，接着我们拿着具体需求去**ConcreteBuilder**中构建具体的手机，并返回给用户。
整个**建造者模式**就是这样的，接下来我们来做下简单的测试：
```
//创建Builder对象
PhoneBuilder miBuilder = new ConcretePhoneBuilder();
//创建管理者
PhoneDirector director = new PhoneDirector(miBuilder);
//生成商品
PhoneProduct product = director.constuct("小米", "骁龙825", "4GB", "1500万像素");
//展示构建结果
textView.setText(product.toString());
```
首先创建**Builder**并传递给**Director**，接下来调用**Director**的构建方法并传递需求，一台手机就构建出来了。
在这个过程中，管理者**Director**完全不知手机道构建细节。代码的扩展性以及可读性都有质的提升。

## 模式变形 ##
在刚刚接触到Builder模式时，我就发现了于我而言，一个非常巨大的应用场景，完美解决一个让我极其难受的问题。
[由于篇幅问题，模式变形我决定新起一篇文章中解释](https://z593492734.github.io/2017/07/11/Builder-Dialog/)。
## 总结 ##
[Demo我已经上传至GitHub，各位看官可自行食用](https://github.com/z593492734/Design-Pattern)。
上面的例子，仅仅是个例子，看起来似乎仅仅是增加了工作量。
但事实并不是这样的，这种写法的扩展性、可读性都是有很大提升的，希望各位看官都可以理解这种**代码思想**。
还有一点是，**建造者模式**与之后要讲到的**工厂模式**类似，他们都是建造者模式，适用的场景也很相似。
一般来说，如果产品的建造很复杂，那么请用**工厂模式**；如果产品的建造**更复**杂，那么请用**建造者模式**。

## 感谢 ##

《Android源码设计模式解析与实战》 何红辉、关爱民 著