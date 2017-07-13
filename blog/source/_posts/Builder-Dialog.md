---
title: 建造者模式的全局Dialog
date: 2017-07-11 16:46:12
tags:
   - 编程思想
   - Android
   - 效果
---
这篇小文将讲述我是如何根据**建造者设计模式**来实现一个全局Dialog。
[如果各位看官还不太了解建造者设计模式，建议可以看一下我的上篇文章。](http://www.jianshu.com/p/07bed15cd13c)

## 背景 ##
在每个项目当中，都会封装一些全局的样式，比如全局Loading、全局Dialog等。
封装这些功能是因为这些控件的使用频率极高。
在我刚接手项目A的时候，项目A也不例外，拥有着全局的Dialog。
刚开始时我尽量写一些侵入性低、仿照率高的代码，避免影响到之前的逻辑。并且学习着如何使用项目A的框架，这个过程我相信大家都是一样的。
当我用到Dialog的时候，我看到了项目A的全局Dialog，详细代码就不说了，给大家看一下项目A全局Dialog中的所有方法：

![](http://7xvzby.com1.z0.glb.clouddn.com/builder_dialog_0.png)
不知各位看官会不会被这么多的构造方法给吓到。
这些构造方法都是随着项目需求的增加而增加的。
大家都知道，一个完整Dialog包括的元素至少应该有：提示图片、标题、描述文字、按钮等。
但是这些元素都不是必须的：
在C页面我弹出的Dialog可能只要做一个温馨提示：一行描述文字外加一个确认按钮。
在D页面我可能弹出的是用户退出登录的二次确认窗口：标题+文字+两个按钮。
那么到这里，各位看官就能明白了，为什么会有那么多的构造方法。
没错，我们在**new KLCustimDialog()**的同时，需要把所有的参数都传入进去。
这种写法的问题以及维护成本之高，我就不做过多描述了，我就简单给大家加个需求：
要求点击Dialog外部不能取消Dialog。
想想这个需求下的构造方法，要添加几个？

## 基本 ##
当我刚看到**建造者模式**的时候，我真是又惊又喜，热血沸腾！
我第一时间想到的就是重构项目A的全局Dialog！
大家都知道，Android中的`AlertDialog`就是使用**建造者模式**来实现的。
在我模仿构思了一波之后，创建Dialog的代码是这样的：
```
Dialog.Builder builder = new Dialog.Builder(context);
builder.setTitle("提示");
builder.setMessage("确认退出登录吗？");
builder.setLeftText("取消");
builder.setRightText("确认");
builder.setOnclickListener(listener);
Dialog dialog = builder.creater();
dialog.show();
```
这样写似乎没有什么问题了。
我们把Dialog的所有元素都默认隐藏，在调用某个元素的填充方法后，我们就将其显示出来。
这样我们就摆脱了无限多的构造方法，完美！
文章到这里就应该结束了？
怎么可能！我都没变形呢！


我见别人的Builder模式都是这样写的：
```
Picasso.with(context)
    .load(url)
    .fit()
    .config(XXX)
    .placeholder(xxx)
    .error(xxx)
    .into(imageView);
```
上述代码是使用**Picasso**来加载url图片。
一行代码完成。
帅不帅，想不想学？
所以我就想能不能用上面作为模板，来实现我们的全局Dialog。
话不多说，说干就干。
## 变形 ##
完整代码我已经上传GitHub，看代码我还是建议各位看官去[我的GitHub](https://github.com/z593492734/Design-Pattern/blob/master/app/src/main/java/com/my/designdemo/builder/dialog/DialogProduct.java)上看，比较整洁。
最终实现的效果如下：
![](http://7xvzby.com1.z0.glb.clouddn.com/dialog_builder_gif.gif)
首先，让我们来回想一下**建造者模式**的组成：
- Product：产品角色
- Builder：抽象的建造者
- ConcreteBuilder：具体的建造者
- Director：指挥者

接下来，我们再把这些成员转化为我们的全局Dialog的成员：
- Product：Dialog就是我们制作出来的产品
- Builder：Dialog参数拼接抽象
- ConcreteBuilder：Dialog参数拼接细节
- Director：所有用到该Dialog的地方都是指挥者，它们决定着Dialog具体样式。

思路有了，下面就开始动手吧，首先我们来创建Dialog的参数封装，里面应该有Dialog所有组成元素：
```
private static class DialogParams {
    private Context context;
    //标题
    private String title;
    //标题字体大小
    private int titleSizeSp;
    //图标资源
    private int imageResource;
    //图标宽
    private int imageWidth;
    //图标高
    private int imageHeight;
    //消息内容
    private String message1;
    //消息内容文字位置
    private int message1Gravity = Gravity.CENTER;
    //点击外部是否可以取消
    private boolean isCanCancel = true;
    //左边按钮内容
    private String leftButtonText;
    //左边按钮颜色
    private int leftBtColor;
    //左边点击事件
    private ConcreteBuilder.ButtonClickLister leftListener;
    //右边按钮内容
    private String rightButtontText;
    //右边边按钮颜色
    private int rightBtColor;
    //右边按钮点击事件
    private ConcreteBuilder.ButtonClickLister rightListener;
}
```
我使用了内部类去实现了整个Dialog，整个Dialog只有一个类，所以所有参数都是private并且没有提供set、get。
并且我为了方便，省略了Builder抽象类，直接构造了Builder抽象类的实现**ConcreteBuilder**：
```
 public static class ConcreteBuilder {
     //持有Product对象
     private DialogParams p;
     
     ConcreteBuilder(Context context) {
         p = new DialogParams();
         p.context = context;
     }
     public ConcreteBuilder title(String text) {
         p.title = text;
         return builder;
     }
     public ConcreteBuilder titleSize(int spSize) {
         p.titleSizeSp = spSize;
         return builder;
     }
     public ConcreteBuilder imageResource(int imageResource) {
         p.imageResource = imageResource;
         return builder;
     }
     public ConcreteBuilder imageWidth(int imageWidth) {
         p.imageWidth = imageWidth;
         return builder;
     }
     public ConcreteBuilder imageHeight(int imageHeight) {
         p.imageHeight = imageHeight;
         return builder;
     }
     public ConcreteBuilder message(String text) {
         p.message1 = text;
         return builder;
     }
     public ConcreteBuilder messageGravity(int gravity) {
         p.message1Gravity = gravity;
         return builder;
     }
     public ConcreteBuilder canCancel(boolean isCanCancel) {
         p.isCanCancel = isCanCancel;
         return builder;
     }
     public ConcreteBuilder leftBt(String text, ButtonClickLister lister) {
         p.leftButtonText = text;
         p.leftListener = lister;
         return builder;
     }
     public ConcreteBuilder leftBtColor(int color) {
         p.leftBtColor = color;
         return builder;
     }
     public ConcreteBuilder rightBtColor(int color) {
         p.rightBtColor = color;
         return builder;
     }
     public ConcreteBuilder rightBt(String text, ButtonClickLister lister) {
         p.rightButtontText = text;
         p.rightListener = lister;
         return builder;
     }
     void clear() {
         p = null;
     }
     public DialogProduct create() {
         return new DialogProduct(p);
     }
     //按钮点击回调
     public interface ButtonClickLister {
         void onClick(DialogProduct dialog);
     }
 }
```
我们会发现每个参数拼接方法都会返回**ConcreteBuilder**,这里是实现一行代码构建Dialog的关键。
参考**Picasso**的书写方式，明显可以看出它没有进行new的行为，说明**with()**一定是静态的，随之**with()**返回的对象也必为静态。
为了实现**Picasso**的书写方式，我们这里也将**ConcreteBuilder**静态，方便实现一句话创建Dialog。
接下来就是Dialog的代码：
```
public class DialogProduct extends Dialog {

    private TextView tvTitle;
    private ImageView ivIcon;
    private TextView tvMessage;
    private TextView tvButtonLeft;
    private TextView tvButtonRight;
    private ImageView viewLine;

    //持有Builder
    private static ConcreteBuilder builder;

    //模仿Picasso的书写方式
    public static ConcreteBuilder with(Context context) {
        if (builder == null) {
            builder = new ConcreteBuilder(context);
        }
        return builder;
    }


    private DialogProduct(DialogParams p) {
        //设置没有标题的Dialog风格
        super(p.context, R.style.NoTitleDialog);

        View contentView = LayoutInflater.from(p.context).inflate(R.layout.dialog_build, null);
        setContentView(contentView);

        tvTitle = contentView.findViewById(R.id.tv_title);
        ivIcon = contentView.findViewById(R.id.iv_icon);
        tvMessage = contentView.findViewById(R.id.tv_message);
        tvButtonLeft = contentView.findViewById(R.id.tv_button_left);
        tvButtonRight = contentView.findViewById(R.id.tv_button_right);
        viewLine = contentView.findViewById(R.id.view_line);

        //控件默认隐藏
        tvTitle.setVisibility(View.GONE);
        viewLine.setVisibility(View.GONE);
        ivIcon.setVisibility(View.GONE);
        tvMessage.setVisibility(View.GONE);
        tvButtonLeft.setVisibility(View.GONE);
        tvButtonRight.setVisibility(View.GONE);
        //构建Dialog
        setTitlText(p.title);
        setTitlTextSize(p.titleSizeSp);
        setImageResource(p.imageResource);
        setImageWidth(p.imageWidth);
        setImageHeight(p.imageHeight);
        setTvMessage(p.message1);
        setTvMessageGravity(p.message1Gravity);
        setCancelableFlag(p.isCanCancel);
        setLeftText(p.leftButtonText, p.leftListener);
        setLeftBtColor(p.leftBtColor);
        setRightText(p.rightButtontText, p.rightListener);
        setRightBtColor(p.rightBtColor);


    }
     /**
     * 设置标题
     *
     * @param title 标题文字
     */
    private void setTitlText(String title) {
        if (TextUtils.isEmpty(title)) {
            return;
        }
        tvTitle.setVisibility(View.VISIBLE);
        tvTitle.setText(title);
    }
    //......省略剩余控件代码
}
```
写完之后，我们来看看这个变形的**建造者模式**的Dialog是如何创建的：
```
DialogProduct.with(this)
        .title("提示")
        .message("您确认退出登录吗？")
        .canCancel(false)
        .leftBtColor(getResources().getColor(R.color.color_0090ff))
        .rightBtColor(getResources().getColor(R.color.color_f96c59))
        .leftBt("取消", new NormalDialog.ConcreteBuilder.ButtonClickLister() {
            @Override
            public void onClick(NormalDialog dialog) {
                dialog.cancel();
            }
        })
        .rightBt("确认", new NormalDialog.ConcreteBuilder.ButtonClickLister() {
            @Override
            public void onClick(NormalDialog dialog) {
                Toast.makeText(BuilderActivity.this, "退出登录成功！", Toast.LENGTH_SHORT).show();
                dialog.cancel();
            }
        })
        .create()
        .show();
```

## 总结 ##
[DialogProduct的代码我已经上传到了GitHub](https://github.com/z593492734/Design-Pattern/blob/master/app/src/main/java/com/my/designdemo/builder/dialog/DialogProduct.java)，各位看官可以自行食用。
这个Dialog现在也是项目A中的全局Dialog，使用起来也非常方便。
这里有几个细节可以和各位看官分享一下：
第一个就是按钮的点击事件设置，我将其与按钮文字内容的设置绑定在一起。因为我认为你设置了按钮，怎么可能会没有点击事件？
第二个就是按钮中间的分割线，是与右边按钮绑定的， 所以当只有一个按钮时，我们应该使用左边的按钮**leftBt**而不是右边的。
基本上就是这样啦！
希望这个Dialog可以给大家带来一些灵感。