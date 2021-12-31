## 涂飞平的博客

```
网名: 网络老鼠
公众号: 网络老鼠技术小屋
角色：程序员、架构师、技术总监
语言：C/Java/Delphi/Python/TypeScript
就职：用友审计
邮箱：dHVmZWlwaW5nQGhvdG1haWwuY29t
```

个人博客采用 `github` 来编写和发布，都是原创内容，记录技术，感想，生活等相关内容，所有见解均来自个人！您可以在这里看到整个博客的源码 [editor on GitHub](https://github.com/tufeiping/tufeiping.github.io/edit/master/README.md)。

文档内容搜索移步 <a href="https://sourcegraph.com/github.com/tufeiping/tufeiping.github.io" target="_blank">这里<a> 🔍

关于个人博客的一些历史记录：

- 2003 年 博客中国开启博客
- 2013 年 转战 `SAE` 平台
- 2015 年 独立部署
- 2020 年 转移到 `github` 平台（所有的内容均同步来自公众号 `网络老鼠技术小屋`，以前历史的博客不再发布了）

<p style="position: relative;float: right;top: -202px;">
<!--
<img src="http://store.tufeiping.cn/Header.jpg" alt="Header.jpg">
-->
<img src="./Header.svg"/>
</p>
<br/>

### 常见浏览器 User Agent 及应用

最近在做 App 二维码下载的功能，需要在后端判断扫描来自 PC，Android 手机还是苹果手机，所以将常见的浏览器 User Agent 记录下来。

[详细...](http://www.tufeiping.cn/2019/UserAgent)

### 扩展 Delphi IDE

Embarcadero Delphi 提供丰富的 API 以便 Delphi 开发人员自定义和扩展自己的 Delphi IDE。本文的目的就是通过介绍其中的部分 API 和示例，并且，还会有部分 TMS 公司提供的部分免费的 IDE 扩展的源码，给大家演示如何编写 Delphi IDE 扩展。

[详细...](http://www.tufeiping.cn/2019/DelphiIDE)

### Maven 简介

Maven 出来很多年了，我大概是在 2008 年，通过 DSpace 开源项目，第一次实际使用 maven 的，在之前，多半是写 ANT 脚本来构建 Java 项目，都这么古典的知识了，为什么还要介绍一下？主要是为了给公司新同事提纲挈领的快速过一遍，不求精通，但希望通过本文能对 Maven 做到心中有数。

[详细...](http://www.tufeiping.cn/2018/Maven)

### 东瀛游记

日本游其实是无意为之，去年底 Twitter 上关注了一个日本朋友，经常看他 tweet，然后突然就想去看看！所以，今年就开始张罗，办理好日本签证，耗时 1 个月，其中半个月是我办理的时间，然后给老婆也办理一个（由于是家属，所以她办理签证要准备的资料就很简单了）。

[详细...](http://www.tufeiping.cn/2019/japantravel)

### 程序技术人员的进阶

这段时间，面试，面谈了些新人和同事，与他们的不断交流中，发现他们都很努力，也想做好这个职业，但由于是刚刚进入这个职业，还是需要了解该如何学习。
带过很多小孩了，基本上，很多次聊这个话题，基本内容都差不多，我想可以简单写下来，以后就直接给其他人看，就能够少说一点，他们也能长期保存，而不至于听完半年后就淡忘了。

[详细...](http://www.tufeiping.cn/2019/ProgramerLife)

### Python 的 Decorator

Python 语言中的 Decorator，也就是常说的装饰器，个人认为是 Python 语言中的亮点之一，在语言层面（从语义上）直接支持装饰器的我所知道的也就是 Python 了，虽然 Javascript 也是支持的，但需要通过技术手段来处理一下（都是将函数赋值给同名的变量以达到相同的效果），而 Python 虽然在原理上是一致的，但其在语言层面直接支持，提供了@运算符来标识（跟 Java 提供的注解采用一样的标识符，而且作用也类似），更直观，更简洁！

[详细...](http://www.tufeiping.cn/2018/pythondecorator)

### 清晨，美丽的园子

所谓`园子`，其实是用友人对`用友产业园`的昵称。坐落在京西的`用友产业园`是用友软件总部，研发基地，总面积达 40 万平方米，有着京西最美的园区风景，每一个到此参观的客户，都对园区风景称赞不已！
今天是五一假期后上班第一天，也许是假期休息得好，很早就醒了，6 点多一点就到公司

[详细...](http://www.tufeiping.cn/2018/Yonyou)

### git https/http 方式下免密登录

在使用 Git 的时候，一般情况下都是使用 http/https 方式，这个种方式虽然比 ssh 免密登录多了一个输入用户名/密码的步骤，但由于其不需要将公钥存入 git server，所以目前是我们部门的推荐（默认）使用方式。

[详细...](http://www.tufeiping.cn/2019/gitauth)

### TinyHttpd 源码分析

TinyHttpd 是一个 C 编写的极小 HTTP 服务器，代码量（包括注释）不到 500 行，但可以基本说明 HTTP 服务器的工作原理，从中也能知道如何进行 Linux 下的 C 编程（这是我看这个代码的主要原因）。

[详细...](http://www.tufeiping.cn/2018/TinyHttpd)

### 如何在 Docker 中安装 DzzOffice

先在官网 http://www.dzzoffice.com/ 上面了解了一下 DzzOffice 依赖的环境和背景，它运行在典型的 LAMP/WAMP 平台上。是否可以在公司内部服务器 Docker 群中加入一个 Docker 容器来专门运行 DzzOffice？经过实际测试，很轻松就完成这个设想，很完美。为了便于大家在 Docker 上面体验 DzzOffice，我将 Docker 工程提交 Github 上面，接下来就根据 Github 上面的工程来一步一步安装 DzzOffice。

[详细...](http://www.tufeiping.cn/2014/dockerdizz)
