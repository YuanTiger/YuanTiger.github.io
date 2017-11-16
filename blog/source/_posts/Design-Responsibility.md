---
title: 提升代码灵活性-责任链模式
date: 2017-11-16 15:54:00
tags:
   - 编程思想
---
## 模式介绍 ##
**责任链模式**是行为设计模式之一。

首先我们从字面去理解**责任链模式**：“责任”指的是一个人应尽的义务、分内应做的事；“链”指的是一个个小环首尾相连，组成的长链条。

为什么是“链”？因为“链”的每一环都是可以拆卸的，它们虽然是环环相扣，但是哪天我不想要中间的某一环，我只需要将其去下来即可，这大大提升了代码的灵活性。

所以**责任链模式**通俗来讲，指的是**一件**事情，有顺序地传递给**一群人**，当第一个人无法处理时，再传递给第二个人去做，直到有人可以解决掉这件事情为止。

## 模式示例 ##
小明是一家技术公司的测试，他除了完成本职工作外，还负责给技术部的小伙伴购买零食，不过零食钱是先由小明自己垫付的，接着到月底拿着发票去找领导报销的流程。

这个月的月底，小明算了一下，一共花费了2450元。

接着小明拿着零食的发票去找组长报销，结果组长说这金额太大了，只有500以下我才有权限签字，你得去找技术总监。

接着小明找到技术总监，总监说2450太多啦，只有2000以下我才有权限签字，你得去找咱们的老板。

接着小明去到老板的办公室，老板看了一下说非常好，技术部的同学们都非常辛苦，以后都多买一些零食，接着麻溜地就签了字，小明顺利完成了此月的零食报销请求。

## 使用场景 ##
通过上述示例，我相信大家已经有些理解**责任链模式**了。

组长、总监、老板针对小明的报销请求，都有自己要做的事情，但是根据小明的报销金额不同，无法第一时间就知道到底是谁来处理这个报销请求。

这就需要小明按照一定的顺序，一个个地去问，直到找到能处理此次报销请求的人为止。

这种情况下，就非常适合使用我们今天要说的**责任链模式**。

通过上面的描述，我们可以得出**责任链模式**的使用场景：
- 一个请求需要多个对象处理，但具体由哪个对象处理需要在运行时动态判断时；
- 需要动态指定一组对象处理一个请求。

## 所含角色 ##
**责任链模式**包含两个主要角色：
- 处理者抽象(Handler)：抽象的处理者，声明一个处理请求的方法，并在其中保持对下一个处理者的引用。
- 处理者实现(HandlerImpl)：处理者抽象的具体实现，对请求进行处理，如果无法处理，则通过下一个处理者的引用将其转发下去。

## 具体代码 ##
针对上述示例和角色描述，我们来将其转化成具体的代码。

首先是处理者的抽象，针对上述示例，我们的抽象的请求处理方法应该为报销：
```
public abstract class Leader {
    //下一个处理者
    private Leader nextLeader;

    /**
     * 处理报销请求
     *
     * @param money 申请报销的金额
     */
    public final void handlerRequest(int money) {
        if (money <= getSelfLimit()) {
            //当申请的金额<=自己的处理额度时,将此次请求消化
            handler(money);
        } else {
            //否则交给下一个处理者来处理
            if (nextLeader != null) {
                nextLeader.handlerRequest(money);
            }
        }

    }

    /**
     * 获取自身的额度，由具体实现来设置
     *
     * @return 自身能处理的额度
     */
    public abstract int getSelfLimit();

    /**
     * 具体处理逻辑在这里实现
     *
     * @param money 申请报销的金额
     */
    public abstract void handler(int money);


    public void setNextLeader(Leader nextLeader) {
        this.nextLeader = nextLeader;
    }

    public Leader getNextLeader() {
        return nextLeader;
    }
}
```
代码还是有些多的，让我们来解释一下这个处理者抽象类：
- nextLeader：指定了当请求无法处理时，下一级的处理者。这是**责任链模式**中，“链”的核心。对外暴露了set、get方法。
- handlerRequest(int money)：注意此方法不是抽象并且是final修饰的，也就是无法重写，无法修改。它其中的逻辑是用来判断此次请求，应该是由自己消化，还是分发给下一个处理者。当你要开始请求时，只需要调用此方法即可。为了适应我们举的示例，这里有一个money参数。这里说一下，每一种设计模式仅仅代表着一种编程思想，具体代码还是要看需求的，如果需求复杂，那么这里分发的逻辑也会复杂、参数也会更多，反之亦然。
- getSelfLimit()：抽象方法，指定处理者的最高处理额度。
- handler(int money)：当此次请求该由自己消化时，具体的消化逻辑在这里实现。

如果还有疑问，不要急，我们先来具体看一个处理者的实现，可能你就会恍然大悟：
组长：
```
public class LeaderGroup extends Leader {

    /**
     * @return 组长报销的处理额度
     */
    @Override
    public int getSelfLimit() {
        return 500;
    }

    /**
     * 当报销金额 <= getSelfLimit() 时，请求会到这里来消化
     *
     * @param money 申请报销的金额
     */
    @Override
    public void handler(int money) {
        Toast.makeText(App.context, "小钱，组长报销:" + money, Toast.LENGTH_SHORT).show();
    }
}
```
组长继承了抽象的处理者，并指定了组长的报销额度以及具体的处理逻辑。
同样的，总监和老板也是一样的道理：
总监：
```
public class LeaderGeneral extends Leader {

    /**
     * @return 总监报销的处理额度
     */
    @Override
    public int getSelfLimit() {
        return 2000;
    }


    /**
     * 当报销金额 <= getSelfLimit() 时，请求会到这里来消化
     *
     * @param money 申请报销的金额
     */
    @Override
    public void handler(int money) {
        Toast.makeText(App.context, "金额挺大，总监报销:" + money, Toast.LENGTH_SHORT).show();
    }
}
```
老板：
```
public class LeaderCEO extends Leader {

    /**
     * @return 老板报销的处理额度
     */
    @Override
    public int getSelfLimit() {
        return Integer.MAX_VALUE;
    }


    /**
     * 当报销金额 <= getSelfLimit() 时，请求会到这里来消化
     *
     * @param money 申请报销的金额
     */
    @Override
    public void handler(int money) {
        Toast.makeText(App.context, "这么多钱，老板报销:" + money, Toast.LENGTH_SHORT).show();
    }
}
```
至此，我们的处理者抽象，以及处理者实现都已经完成了，下面我们来具体模拟一下报销流程：
```
//获取输入框中输入的金额
String money = et_money.getText().toString();
if (TextUtils.isEmpty(money)) {
    Toast.makeText(this, "请输入金额", Toast.LENGTH_SHORT).show();
    return;
}
//组长实例
Leader group = new LeaderGroup();
//总监实例
Leader general = new LeaderGeneral();
//CEO实例
Leader CEO = new LeaderCEO();
//组长的下一级是总监
group.setNextLeader(general);
//总监的下一级是CEO
general.setNextLeader(CEO);
//由组长优先处理报销
group.handlerRequest(Integer.valueOf(money));
```
上面的测试代码逻辑也很清晰：
1. 获取到具体的报销金额
2. 首先创了组长、总监、CEO的实例
3. 按照组长 -> 总监 -> 老板 的报销顺序将其进行排序
4. 由组长优先处理报销请求，当组长无法处理时，会由组长实例中的下一个处理者来处理。

当我们需求发生变化，比如组长的报销金额变成了1000、比如现在组长没有报销权限了等等，你会发现非常地好维护。

这里我们假设总监无法报销了，当组长无法报销时，直接找到CEO来报销：
```
Leader group = new LeaderGroup();
Leader CEO = new LeaderCEO();

group.setNextLeader(CEO);

group.handlerRequest(Integer.valueOf(money));
```
我们仅仅是将组长的下一个处理者改为老板即可，就可以完美删除掉总监的存在。

这种“链”式的写法，完美解耦了请求者和处理者，提高代码了灵活性。

## 总结 ##
到这里，一个最基本的**责任链模式**就完成了。

[代码已经上传至GitHub](https://github.com/YuanTiger/Design-Pattern)。

**责任链模式**的优点已经说过了。

缺点就是处理者太多的话，必定会影响性能，尤其是在一些递归调用中，使用时一定要注意。

## 感谢 ##

《Android源码设计模式解析与实战》 何红辉、关爱民 著




