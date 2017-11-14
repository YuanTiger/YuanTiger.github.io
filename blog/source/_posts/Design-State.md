---
title: 随时灵活变化-状态模式
date: 2017-11-14 16:39:52
tags:
   - 编程思想
---
## 模式介绍 ##
**状态模式**的结构和[策略模式](http://www.jianshu.com/p/5c1f67bc2d66)几乎一模一样。

但是它们两者的目的、本质却完全不同。

[策略模式](http://www.jianshu.com/p/5c1f67bc2d66)的行为是彼此独立，相互替换的。回想之前举的价格计算器，我们出行时想使用地铁，则使用地铁的价格计算器，如果使用出租车，则使用出租车的价格计算器......

而**状态模式**的行为则是平行的，不可替换的。**状态模式**更像是被封装在对象内部的一个东西，当对象的状态发生变化时，其行为也要发生变化。

## 使用场景 ##
- 一个对象的行为需要根据状态发生变化，尤其是在运行时状态发生变化，行为需要跟着一起变化时；
- 代码中有大量判断语句(if、switch)，且这些语句依赖该对象的状态。

## 模式角色 ##
- State：状态接口，定义所有行为的抽象。
- ConcreteStateA、ConcreteStateB：某个状态的具体行为实现，一个状态对应一个具体行为实现。

## 模式示例 ##
相信大家在开发中，只要是动态App(和服务器有交互)，都会有登录的需求。

针对这个需求，我们可以得出两个状态：登录状态、未登录状态。

接着我们可以想象一些行为：
- 登录
- 退出登录
- 查看账户金币
- 赚取金币
- 金币兑换道具

针对上述行为，在登录状态和未登录状态下，所做的事情是完全不同的：
1. 登录状态：
   - 登录：提示用户已经登录过了，请勿重复登录
   - 退出登录：清除用户缓存并提示用户退出登录成功
   - 查看账户金币：从服务器查询用户余额
   - 赚取金币：跳转到赚金币页面
   - 金币兑换道具：跳转到兑换道具页
2. 未登录状态：
   - 登录：判断用户输入的账号密码，正确提示用户登录成功。
   - 退出登录：提示用户已经退出过了。
   - 查看账户金币：提示用户请先登录
   - 赚取金币：跳转到赚金币页(这里也可以提示用户登录，具体看产品定的登录时机)
   - 金币兑换道具：提示用户请先登录

想必看完这个示例，大家已经对**状态模式**有了一些自己的见解。

这种情况下，非常适合使用**状态模式**。

如果项目中，还是通过判断语句来进行状态的区分，就说明这个项目没有很好地应用**状态模式**。

下面我们就来将上述案例转化成代码。

首先是状态行为的抽象，该抽象包含了所有的行为：
```
public interface UserState {

    //登录
    void login();

    //登出
    void logout();

    //查询余额
    void seeMoney();

    //赚金币
    void earnMoney();

    //兑换道具
    void exchange();
}

```
针对我们上面描述的示例，有以上5种行为，我们将其进行了抽象。

接着来思考行为的具体实现：登录状态的具体实现、未登录状态的具体实现。

具体做起来也很简单，就是创建两个类去实现我们的行为抽象，在各自的实现下，去实现该状态下，应该做的事情。

首先来看登录状态：
```
public class LoginStateImpl implements UserState {
    @Override
    public void login() {
        Toast.makeText(App.context, "您已经登录了，无需登录！", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void logout() {

        Toast.makeText(App.context, "退出登录成功！", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void seeMoney() {
        Toast.makeText(App.context, "您目前有100金币", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void earnMoney() {
        Toast.makeText(App.context, "跳转到-赚金币", Toast.LENGTH_SHORT).show();
    }

    @Override
    public void exchange() {
        Toast.makeText(App.context, "跳转到-兑换道具", Toast.LENGTH_SHORT).show();
    }
}
```
根据示例需求，我实现了登录状态下，这5种行为的具体实现。

未登录状态也是同理，这里就不列举代码了，可以直接查看[GitHub](https://github.com/YuanTiger/Design-Pattern)。

接下来我们来看一下测试代码：
```
//默认是未登录状态
private UserState userState = new LogutStateImpl();

@Override
public void onClick(View view) {
    switch (view.getId()) {
        case R.id.bt_login://登录行为
            userState.login();
            //核心：更换用户状态
            userState = new LoginStateImpl();
            break;
        case R.id.bt_logout://退出登录行为
            userState.logout();
            userState = new LogutStateImpl();
            break;
        case R.id.bt_see_money://查看余额行为
            userState.seeMoney();
            break;
        case R.id.bt_earn_money://赚取金币行为
            userState.earnMoney();
            break;
        case R.id.bt_exchange://兑换行为
            userState.exchange();
            break;
    }
}
```
这里也是利用多态的特性，根据状态的变化，注入不同的实现。

至此，我们已经使用**状态模式**成功实现了上述需求，并且去除了重复、杂乱的判断语句，体现出了**状态模式**的精髓。

## 总结 ##
代码已经上传至[GitHub](https://github.com/YuanTiger/Design-Pattern)，可以下载查阅。

其实看到这里，各位看官想必已经了解了状态模式。

**状态模式**的关键点在于：不同状态下、同一行为的不同响应。

**状态模式**是为了优化代码结构而产生的。我们通过if-else其实可以完美判断用户的登录状态，但是这种实现使得逻辑与具体行为耦合在一起，后期难以维护。

**状态模式**就是为了消除这种丑态，实现逻辑与行为的解耦。

当然并不是所有的if-else都适合使用**状态模式**，具体是否使用，还是由你来决定。
