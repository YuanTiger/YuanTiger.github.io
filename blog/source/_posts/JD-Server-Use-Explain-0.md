---
title: 京东云主机使用(0)-搭建简单网页（macOS）
date: 2017-08-23 11:37:13
tags:
   - 服务器
   - SSH
---
## 前言 ##
在郭霖大神的带领下，我花了一元钱入手了2个月的京东云主机，也就是个人服务器。

这是我人生第一台服务器，多么值得纪念。。。。。。

[入手地址在这里](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650240867&idx=1&sn=d56fceac5b343ccc4a90cd422c490df0&chksm=8863860cbf140f1add1f86fab6267b28e0173350e571ac7192f9a794e2526cadf6dab765d8e2&scene=38#wechat_redirect)

一直不买的原因也是因为自己的Android水平没有达标，不想去学其他方面的知识而分心。

其实很容易发现这他喵的就是一个不想学习的借口罢了！

更容易发现这明显是没钱买吧！

![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/drop_table.jpeg)

所以趁此机会，入手了2个月服务器来尝鲜。名额有限，说不定已经没有了。。。

购买流程就不说了，服务器系统选择的是Ubuntu 16.04 64位。

接下来的使用状况都是围绕着Ubuntu 16.04 64位展开的。

## 登录云主机 ##
郭霖大神推荐了两款软件用于控制服务器 和 上传下载服务器文件:Xshell和Xftp。

但是两款软件都是Windows系统的，没有macOS系统。

如果你是Windows系统的，可移步郭霖大神的[搭建教程](https://mp.weixin.qq.com/s?__biz=MzA5MzI3NjE2MA==&mid=2650240867&idx=1&sn=d56fceac5b343ccc4a90cd422c490df0&chksm=8863860cbf140f1add1f86fab6267b28e0173350e571ac7192f9a794e2526cadf6dab765d8e2&scene=38#wechat_redirect)，相对比较简单。

那么如何在macOS系统下操作服务器呢？

在[京东云的帮助中心](http://www.jcloud.com/help/detail/360/isCateLog/1)中，macOS系统的登录方式有两种：一种是VNC登录，一种是SSH密钥登录。
#### VNC登录 ####
VNC登录是京东云为用户提供的一种通过Web浏览器连接服务器的方式。

很简单，就是在京东云的控制台点击远程连接即可。

接着打开了Ubuntu 16.04 64的控制台，需要先进行登录，用户名为root，密码发送到了你的邮箱和手机当中。
![登录面板](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_0.png)

如果想要修改密码，可在控制台-操作 进行修改。修改完成后记得重启生效。

输入完成并正确就登录上了服务器，非常简单。
![登录成功](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_1.png)

不过使用VNC登录的场景很少：

- 查看云服务的启动进度

- 无法通过其他登录方式登录时，才使用VNC来登录服务器

所以这种登录方式，体验体验即可，并不实用。

并且它不支持复制粘贴、不支持文件上传，而且是单点登录，使用起来简直是折磨。

#### SSH密钥登录 ####
京东云帮助中心提供了[SSH创建和登录教程](http://www.jcloud.com/help/detail/346/isCateLog/1)。

成功设置SSH密钥后，我们就可以不使用VNC登录，直接在Mac的命令行就可以进行服务器的登录。

下面我们来一步一步设置SSH密钥：

什么是SSH密钥？

就我的理解而言，它是一种网络通讯协议，主要用于计算机之间的加密登录。

使用SSH登录的具体流程如下：

![SSH密钥登录](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_2.png)

可以看出一个SSH串要提供给服务器和本机，当SSH串匹配成功后，就可以实现免密登录。

这样的优点就是当登录请求被恶意拦截时，密码也不会泄露。

接下来，我们就要生成SSH密钥，并保存到本机和服务器。

要说一句的是，SSH密钥登录很多地方都有用到，比如GitHub。

如果你的电脑已经有SSH密钥，那么直接使用这个即可。在我的理解下，一台电脑只能有一个SSH密钥。

具体的SSH成功流程可参考[GitHub官方教程](https://help.github.com/articles/connecting-to-github-with-ssh/)。

 在这里我也简单罗列一下SSH密钥的生成步骤：

1.校验本机是否已经生成SSH密钥：
```
ls -al ~/.ssh
```
如果输出了
```
id_dsa.pub
id_ecdsa.pub
id_ed25519.pub
id_rsa.pub
```
则代表已经生成过，直接跳过第二步，执行第三步。

2.生成SSH密钥。如果已经生成跳过。
```
//注意修改最后的E-mail地址
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
执行完成后，会让输入保存路径，直接按下回车，使用默认路径进行SSH密钥的保存就可以。

接着会提示你输入该SSH的密钥密码，可以为空，直接回车，想设置的同学也可以进行设置。

该SSH密钥密码用于第一次使用SSH时的校验，并可以在SSH密钥的配置文件中关闭SSH密钥密码校验。

我是设置的，更多细节大家可以自己去查阅一些资料。

3.复制SSH密钥。
```
pbcopy < ~/.ssh/id_rsa.pub
```
使用该命令后，你的粘贴板内容就会变成SSH密钥。

这次我们要将SSH密钥上传到我们自己的服务器里。

打开京东云的控制板，添加SSH密钥：

![京东云添加SSH密钥](http://7xvzby.com1.z0.glb.clouddn.com/jd_server_use_3.png)

接着点击完成，Over。

4.测试SSH密钥。
使用SSH密钥登录也非常简单。
打开我们Mac的命令行输入：
```
ssh user@xxx.xxx.xxx.xxx
```
user为用户名，我们的用户名为root。@之后为IP地址，比如：
```
ssh root@116.196.93.148
```
接着会提示输入用户输入服务器的登录密码，正确后就可以登录成功。

如果失败，建议按照[京东云帮助中心教程](http://www.jcloud.com/help/detail/346/isCateLog/1)，走一遍。

## 简单网页搭建 ##
我们先为我们的服务器下载一个服务器，这里使用郭神用的apache2。

apache2是专门用来显示静态网页的服务器程序。

在登录服务器成功后输入下面命令：
```
sudo apt-get install apache2
```
接着输入Y完成安装。

之后打开浏览器，输入我们服务器的IP，可以看到下面效果：
![](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_4.png)

接着我们来替换这个html文件样式。

它在我们服务器的地址是：/var/www/html/index.html

我们只要自己写一个简单的静态Html文件，然后上传服务器覆盖掉它即可。

这里我们直接拿着郭神的简单html来做示范：
```
<!Doctype html>
<html>
  <head>
    <title>京东云测试</title>
    <style>
      body{text-align:center}
    </style>
  </head>
  <body>
    <h1>欢迎来到郭霖的京东云主页</h1>
    <p>
      点击
      <a href="http://guolin.tech">这里</a>
      跳转到我的博客
    </p>
  </body>
</html>
```
将该文件保存为index.html。

接着我们将该文件上传至服务器,这里有一篇[mac向服务器上传文件的教程](http://www.jianshu.com/p/1afd25e7459d)。非常好用。
上传命令：
```
//注意将yourUsername修改为你的mac用户名
//并且我的文件保存在桌面Desktop。
put /Users/yourUsername/Desktop/index.html /var/www/html
```
按照上述步骤后，我们成功将index.html上传至服务器并覆盖。
刷新我们的网页，可以看到下面效果：
![效果](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_5.png)

。。。。。。。

![](http://7xvzby.com1.z0.glb.clouddn.com/gaoxiao/drop_table.jpeg)

为什么显示源码！？

因为Mac的记事本以.html结尾时，会将内容格式化成文本，不做代码显示。

解决也很简单，[这篇文章](https://www.zhihu.com/question/21887357)。

解决后重新执行上传代码，重新刷新页面，效果如下：
![效果](http://7xvzby.com1.z0.glb.clouddn.com/jd_server_6.png)

简单查阅后，在head中添加如下代码即可：
```
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
```
接着再次执行文件上传，再次刷新页面，效果如下：
![效果](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_7.png)


## 总结 ##
至此，一个非常简单的静态网页的个人博客便搭建完成了！

写出这么个静态网页，带上这篇博客的出炉，一共耗时2天，走的弯路没有描述。

其中包括Linux命令行控制、SSH密钥理解等，都是新知识，于我而言还是有很大提升的。

以后如果用这个服务器，搭建一个动态的个人博客。

想想还有些小激动呢！