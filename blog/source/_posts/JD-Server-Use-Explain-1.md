---
title: 京东云主机使用(1)-使用Java+Tomcat开始开发
date: 2017-08-24 16:40:03
tags:
   - 服务器
   - SSH
---
## 前言 ##
[上一篇](http://www.jianshu.com/p/7493b1b98c1c)介绍了京东云主机的登录和简单的静态网页配置。

这一篇主要讲述服务器开发环境的配置和部署。

首先，实现服务器开发的语言特别多：Java、PHP、Node.JS、Python等。

这里我选择的是Java+Tomcat+IDEA。

所以本篇文章是围绕着这三者展开的。

并且我们的服务器系统是：Ubuntu 16.04 64。 没看[上一篇](http://www.jianshu.com/p/7493b1b98c1c)的看官需要了解。
## 环境配置 ##

#### Java环境配置 ####
首先我们先输入用户名密码登录我们的服务器：

![登录成功](http://7xvzby.com1.z0.glb.clouddn.com/server/jd_server_use_1.png)

接下来，我们先删除之前下载的apache2，这里我们已经用不到了：
```
sudo apt-get remove apache2
```
现在我们开始下载JDK，先切换目录：
```
cd /usr/local/
```
下载JDK：
```
sudo wget http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-x64.tar.gz
```
这个http链接对应的是写本篇文章时，最新版本的JDK地址。

下载完成后，将其解压：
```
tar -xvzf jdk-8u144-linux-x64.tar.gz
```
解压完成后，删除压缩包：
```
rm jdk-8u144-linux-x64.tar.gz
```
到这里，下载步骤就算完成了，接着开始环境变量的配置：
```
vim /etc/profile
```
Vim是Linux文件编辑模式，该命令会打开文件并可进行编辑。

该命令的功能非常强大，如果之前你没有见过该命令，这次可以先稍微了解一下。

[如果非要现在学习vim，可以参考此文。](http://pizn.github.io/2012/03/03/vim-commonly-used-command.html)

这里我简单介绍一下该命令。

Vim命令有三种模式：普通模式、编辑模式、命令模式。

顾名思义，想要编辑必须进入编辑模式，想要进行保存或是退出就要进入命令模式，平时浏览时处于普通模式即可。

这里我们以刚才执行的命令开始进行讲解。

在执行完`vim /etc/profile`后，会打开/etc/profile文件，此时默认处于普通模式。

我们只需一直往下滑动，或是使用Ctrl+F/D，滑动到文件底部。

接着将光标移动至内容的最下方，点击i/c/o进入编辑模式，复制以下内容：
```
JAVA_HOME=/usr/local/jdk1.8.0_144
JAVA_BIN=/usr/local/jdk1.8.0_144/bin
JRE_HOME=/usr/local/jdk1.8.0_144/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JAVA_BIN JRE_HOME PATH CLASSPATH
```
如果各位看官修改了安装路径，这里注意要匹配。

复制完成后，点击Esc，即可退出编辑模式。

接着输入冒号（Shift+；），进入命令模式。

输入wq，然后回车，进行文件的保存并退出Vim模式。

至此，Java的环境变量应该已经配置成功，使用命令来刷新环境：
```
source /etc/profile
```
接着输入：
```
java -version
```
如果效果如下：

![Java环境测试](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_0.png)

代表环境变量已经配置成功。

如果失败，请检查环境变量的路径是否正确，或是JDK是否安装成功。

#### Tomcat环境配置 ####
```
cd /usr/local/
```
下载Tomcat压缩包:
```
sudo wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.5.20/bin/apache-tomcat-8.5.20.tar.gz
```
该包的版本是Tomcat8.5.20，如果不适合你，可以去[Tomcat下载页](http://archive.apache.org/dist/tomcat)自行选择。

接着进行解压：
```
tar zxvf apache-tomcat-8.5.20.tar.gz
```
解压完成后，这个文件的名字也太长了，我们重命名一下：
```
mv apache-tomcat-8.5.20 /usr/local/tomcat
```
接着拷贝catalina.sh
```
cp -p /usr/local/tomcat/bin/catalina.sh /etc/init.d/tomcat
```
开始配置环境：
```
vim /etc/init.d/tomcat
```
操作方式和之前描述的一样，在开头直接添加以下内容：
```
JAVA_HOME=/usr/local/jdk1.8.0_144
CATALINA_HOME=/usr/local/tomcat

chmod 755 /etc/init.d/tomcat
chkconfig --add tomcat
chkconfig tomcat on
```
至此，Tomcat的环境变量配置工作，已经完成。

下面我们来启动Tomcat：
```
service tomcat start
```
验证启动是否成功：
```
ps -ef|grep tomcat
```
如果输出内容如下,则代表启动成功：
![Tomcat启动成功](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_2.png)

如果输出内容如下，则代表失败：
![Tomcat启动失败](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_1.png)

启动失败的原因基本上都是因为环境配置的问题，请仔细检查路径配置。

启动成功后，在浏览器输入IP://8080,比如：
```
116.196.93.148:8080
```
即可看到如下效果：
![Tomcat运行效果](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_3.png)

有些人说，我不想输入端口号，怎么办？
```
vim /usr/local/tomcat/conf/server.xml
```
执行上述命令打开Tomcat的配置文件，找到如下内容：

![server修改端口号](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_4.png)

进入编辑模式将port 改为 80 即可。

如果你还买了域名，想要监听域名，找到如下内容：

![server修改默认地址](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_5.png)

进入编辑模式将name改为自己的域名即可。

修改完成后进入命令模式输入wq保存并退出。

接着重启Tomcat：
```
service tomcat stop
service tomcat start
```
之后再访问的话，就不用带端口号了。

如果你修改了Host-name，记得使用你修改的域名。

至此，我们服务器的环境搭建工作就告一段落了。
## HelloWeb ##
接下来我们就要开始开发我们的Web项目了！

这里我下载了[IntelliJ IDEA](https://www.jetbrains.com/idea/download/#section=mac)作为我的开发软件。

这里注意要下载商业版，也就是Ultimate版。

Community版是不支持Web开发的。

下载完成后，我们需要为本机也装上Java和Tomcat。

接着我们就可以开始开发HelloWorld了！

具体的下载和开发流程，我这里就不介绍了，[参考此文](http://www.cnblogs.com/carsonzhu/p/5468223.html)。

我就是按照一步一步来进行开发的。

按照上述步骤开发完成后，我们的HelloWorld就已经在本机完成了。

效果如下：

![localhost:8080](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_6.png)

接着我们要将该项目部署到我们自己的服务器上，让所有人都可以访问！

最基本的方法就是将项目达成War包，将War包上传至Linux服务器的Tomcat/webapps文件夹中。

之后在访问时，Tomcat会自动将War包进行解压。

传输的方法，可以参考上次说到的[Mac向服务器上传文件](http://www.jianshu.com/p/1afd25e7459d)。

Windows的话可以使用Xshell。

这里我们就不做过多介绍了，因为这种方式明显过于繁琐，每次修改项目都要重新生成War包并上传服务器。

我希望我们可以像是在本机调试一样，修改完成后点击运行就可以看到效果。

这就是我们接下来要做的事情。

## Hello 热部署 ##
我们首先登录自己的Linux服务器，切换到自己的Tomcat的bin目录下：
```
 cd /usr/local/tomcat/bin/
```
接下来打开VIM模式，开始编辑**catalina.sh**文件：
```
vim catalina.sh
```
在文件的100行，也就是`# OS specific support.  $var _must_ be set to either true or false.`之上，添加如下代码：
```
export CATALINA_OPTS="-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=9999 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.authenticate=false 
-Djava.rmi.server.hostname=116.196.93.148"

export JAVA_OPTS="-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false"
```
其中要注意的是，hostname为自己的IP地址，两个port设置为和本机不冲突的端口号即可，注意此端口号不是Tomcat的访问端口，要进行区分，比如我设置的就是9999，你也可以选择使用它。

添加修改完成后保存退出，接下来要对自己的IP地址进行映射。先查看自己的用户名：
```
uname -n
```
将显示出来的用户名进行复制。

接下来打开hosts文件：
```
 vim /etc/hosts
```
添加如下代码：
```
127.0.1.1      复制的用户名
::1     复制的用户名
```
注意不用删除和注释其他映射，只需添加即可。

映射完成后，我们需要重启Tomcat：
```
service tomcat stop
```
执行完后请确认是否关闭成功：
```
ps -ef|grep tomcat
```
如果输出

![Tomcat关闭失败](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_2.png)

则代表关闭失败，执行下句命令强制关闭：
```
//port为端口号，像上述图片中，端口为2392
kill -9 port
```
接下来启动Tomcat，因为我们修改了**catalina.sh**，所以我们不能使用**startup.sh**来启动Tomcat，需使用：

```
//注意此时的目录还是在Tomcat/bin目录下
// > /dev/null 2>&1 &  的含义是不显示启动日志
sh catalina.sh run > /dev/null 2>&1 &
```
确认启动成功：
```
ps -ef|grep tomcat
```
启动成功后，我们Linux服务器的配置就结束了！

我们回到IDEA中，进行IDEA的配置。

首先打开Configurations：

![Configurations](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_7.png)

接着创建远端Tomcat：

![创建远端Tomcat](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use8.png)

接着开始进行远端Tomcat的配置：

![远端Tomcat配置](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_9.png)

SFTP协议配置：

![SFTP协议配置](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_10.png)

接着添加部署项目：

![添加项目](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_11.png)

点击OK进行保存，接着我们选择使用远端Tomcat来部署并运行项目：

![运行项目](http://7xvzby.com1.z0.glb.clouddn.com/jd1_server_use_12.png)

如果出现以下效果：

![运行失败](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_13.png)

代表连接远端失败，请仔细检查你的配置，此过程比较耗时。

运行成功Log如下：

![成功Log](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_14.png)

效果如下：

![成功效果](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_15.png)

注意上图访问的IP，是我们Linux服务器的地址，也就代表我们成功将项目部署到了我们的服务器上。

接着我们修改index.jsp，再次运行项目：

![热部署效果](http://7xvzby.com1.z0.glb.clouddn.com/server/jd1_server_use_16.png)

可以看到我们修改的效果已经成功部署到服务器中。

上文中的IP就是我的服务器IP，大家也可以访问的。


## 总结 ##
至此，我们已经成功使用IDEA+Tomcat+Java实现了项目的开发以及热部署。

接下来就比较自由了，自己的项目就由自己去写了。

可以搭建个人博客，可以搭建资源共享网页等等。

各有各的风格，大家可以自行学习后台开发，搭建过程就不准备再写博客了。

碰巧今天是七夕节，今天看文章的人都是敬业的程序员！

噗哈哈哈，祝大家七夕节开心快乐！

## 感谢 ##
[搭建JDK+Tomcat](http://wiki.jikexueyuan.com/project/linux/tomcat.html)

[Mac向服务器上传文件](http://www.jianshu.com/p/1afd25e7459d)

[idea部署项目到远程tomcat](http://blog.csdn.net/tianjun2012/article/details/52795202)

[Linux VIM命令使用](http://pizn.github.io/2012/03/03/vim-commonly-used-command.html)
