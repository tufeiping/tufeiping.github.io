## 涂飞平的博客

~~~
涂飞平
网名: 网络老鼠
角色：程序员、架构师、技术总监
语言：C/Java/Delphi/Python/TypeScript
就职：用友审计
~~~

个人博客采用 `github` 来编写和发布，都是原创内容，记录技术，感想，生活等相关内容，所有见解均来自个人！您可以在这里看到整个博客的源码 [editor on GitHub](https://github.com/tufeiping/tufeiping.github.io/edit/master/README.md)。

关于个人博客的一些历史记录：

- 2003年 博客中国开启博客
- 2013年 转战 `SAE` 平台
- 2015年 独立部署
- 2020年 转移到 `github` 平台（所有的内容均同步来自公众号 `网络老鼠技术小屋`，以前历史的博客不再发布了） 

<p style="text-align: center;"><img src="https://raw.githubusercontent.com/tufeiping/tufeiping.github.io/master/assets/Header.jpg" alt="Header.jpg"></p>

### 常见浏览器User Agent及应用

最近在做App二维码下载的功能，需要在后端判断扫描来自PC，Android手机还是苹果手机，所以将常见的浏览器User Agent记录下来。

[详细...](http://www.tufeiping.cn/2019/UserAgent)

### 扩展Delphi IDE

Embarcadero Delphi 提供丰富的API以便Delphi开发人员自定义和扩展自己的Delphi IDE。本文的目的就是通过介绍其中的部分API和示例，并且，还会有部分TMS公司提供的部分免费的IDE扩展的源码，给大家演示如何编写Delphi IDE扩展。

[详细...](http://www.tufeiping.cn/2019/DelphiIDE)

### 东瀛游记

日本 游其实是无意为之，去年底Twitter上关注了一个 日本 朋友，经常看他tweet，然后突然就想去看看！所以，今年就开始张罗，办理好 日本 签证，耗时1个月，其中半个月是我办理3年多次签，然后给老婆也办理一个（由于是家属，所以她办理签证要准备的资料就很简单了）。

[详细...](http://www.tufeiping.cn)


### Python的Decorator

Python语言中的Decorator，也就是常说的装饰器，个人认为是Python语言中的亮点之一，在语言层面（从语义上）直接支持装饰器的我所知道的也就是Python了，虽然Javascript也是支持的，但需要通过技术手段来处理一下（都是将函数赋值给同名的变量以达到相同的效果），而Python虽然在原理上是一致的，但其在语言层面直接支持，提供了@运算符来标识（跟Java提供的注解采用一样的标识符，而且作用也类似），更直观，更简洁！

[详细...](http://www.tufeiping.cn/2018/pythondecorator)

### git https/http方式下免密登录

在使用Git的时候，一般情况下都是使用http/https方式，这个种方式虽然比ssh免密登录多了一个输入用户名/密码的步骤，但由于其不需要将公钥存入git server，所以目前是我们部门的推荐（默认）使用方式。

[详细...](http://www.tufeiping.cn/2019/gitauth)

### TinyHttpd源码分析

TinyHttpd是一个C编写的极小HTTP服务器，代码量（包括注释）不到500行，但可以基本说明HTTP服务器的工作原理，从中也能知道如何进行Linux下的C编程（这是我看这个代码的主要原因）。

[详细...](http://www.tufeiping.cn/2018/TinyHttpd)


### 如何在Docker中安装DzzOffice

先在官网 http://www.dzzoffice.com/ 上面了解了一下DzzOffice依赖的环境和背景，它运行在典型的LAMP/WAMP平台上。是否可以在公司内部服务器Docker群中加入一个Docker容器来专门运行DzzOffice？经过实际测试，很轻松就完成这个设想，很完美。为了便于大家在Docker上面体验DzzOffice，我将Docker工程提交Github上面，接下来就根据Github上面的工程来一步一步安装DzzOffice。

[详细...](http://www.tufeiping.cn)
