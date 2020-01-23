## Maven简介

> 原创 涂飞平  `2018-10-19`

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven1.png" alt="maven1.png"></p>

Maven出来很多年了，我大概是在2008年，通过DSpace开源项目，第一次实际使用maven的，在之前，多半是写ANT脚本来构建Java项目，都这么古典的知识了，为什么还要介绍一下？主要是为了给公司新同事提纲挈领的快速过一遍，不求精通，但希望通过本文能对Maven做到心中有数。

### Maven与其他构建工具的不同

#### ANt工具

早期的项目，我都是写Ant来构建的，这里不光是Java项目，实际上，在上家公司，我们的Delphi项目也是依靠Ant来构建系统的，为什么要用ANt构建项目？
这里需要灌输的一个理念：能够自动化的，一定不要手动做，哪怕就点一下鼠标，都不能！一定要有工具化的理念，只有这个概念扎根脑海，你才会在做系统功能的同时，考虑提供各种自动化（测试）接口或者工具，才会在做UI的同时，去提供CLI版本的实现，这样就能倒逼自己将项目模块化，将功能与UI和CLI呈现层面的东西隔离开。

ANt是什么？ANt你可以认为是一个以xml语言来定义工作的自动化工具，与shell脚本或者bat脚本定义批处理自动化工具是类似的。
可以对比两者

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven2.png" alt="maven2.png"></p>

有了Ant，为什么还需要Maven？这个疑问在我第一次接触Maven的时候也有过，总觉得老外为什么喜欢重新做轮子！但真到你使用Maven之后，你会发现，Maven与Ant有太多的不同，正是这些不同，导致了目前Java体系中，Maven几乎就是事实标准的构建工具，而且，其影响力远超Java体系，npm等新语言的构建工具，都能看到Maven设计思想的影子。

Maven自身的定位是个工程(项目)管理工具！从其配置文件pom.xml(Project Object Model)命名就能明白其定位了。

1、Maven采用COC（约定优于配置）的思想来标准化项目模板，其构建仅针对支持的模板；这与Ant不一样，Ant太灵活，可以迁就各种项目目录格式，甚至，Java文件可以放在不同的Folder中，只要编译后Copy到一个Classes里面即可，看起来虽然很强，但很乱，每个项目，每个模块，甚至每个开发小组，都可以定义自己的组织方式和目录结构，然后丢出一个build.xml（ant的构建脚本定义文件），这对于大型项目就是一个灾难！Maven从一开始就杜绝这种项目管理方式，采取了约定项目结构的方式，每种项目类型，都有其固定的目录结构，后续maven的命令，将根据项目不同，针对标准的目录结构做编译，测试，打包，发布等工作。

2、软件仓库（中央/私有）
Ant仅仅是个自动化工具，它不负责依赖的管理，我们需要自己将项目依赖的jar包从官网或者其他途径下载下来，然后放在Classpath(Libs)里面，这会带来以下几个问题：
2.1、版本依赖难于控制，可能出现Libs里面出现几个相同的，版本不同的Jar包，你根本不知道应该删除哪个；
2.2、依赖随着项目推进，无效依赖Jar越来越多，重构代码后，你引入新的依赖，但却不敢删除旧的依赖（大型项目尤其如此），这样会导致Libs里面的jar越来越多；
2.3、项目代码分享必须带Jars，由于手工管理依赖包，在项目分享（如提交到git或者svn仓库）的时候，你必须将这些依赖包也一并提交，导致代码仓库耗费存储容量巨大。

Maven采取仓库的思想（与AppStore和Android应用商店一样），所有的依赖都放在仓库里面，项目所有的依赖关系由pom.xml统一定义，在需要的时候（如编译，测试，打包等）再从仓库拉取。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven3.png" alt="maven3.png"></p>

将所有的依赖通过定义方式，统一处理，每个项目的构建环境能做到一致，并且将依赖，依赖的依赖统一处理，在代码管理方面，再也不需要将依赖jar包拷贝来拷贝去了，可以避免及解决上述三个问题。

#### Maven中的配置

maven最重要的配置文件就两个

- settings.xml

settings.xml有两个地方可以存在，一个是$M2_HOME/conf的settings.xml，这个是全局的maven配置文件，一个是$HOME/.m2的settings.xml，是当前用户的maven配置，是可选的。
在maven运行的时候，如果存在个人配置文件settings.xml，会override全局settings设置。

settings.xml里面比较重要的几个配置项：
localRepository：配置本地仓库的地址（本机目录），默认是$HOME/.m2
mirrors：镜像服务器配置，如果有私服，就需要配置这个选项，其实镜像库在工作方式上，你可以认为是网络上的localRepository，它遇到没有的依赖包，也会从其配置的中央仓库或者上级仓库拉取（两图相似）。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven4.png" alt="maven4.png"></p>

settings.xml每个配置项都有详细的注释，并提供了一个配置模板，大家可以自己看看。
在各个IDE（Eclipse，IDEA，NetBeans）中，maven插件安装好之后，第一件事情就是配置其settings.xml指向，以便IDE能获取本地Maven的配置（如果没有设置，IDE的Maven插件一般会用默认配置）

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven5.png" alt="maven5.png"></p>

-  pom.xml

第二个重要的配置文件是pom.xml，这是每个项目的配置文件。
pom.xml是maven管理项目的关键（所有配置均在这里），最常见的pom.xml如下：

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven6.png" alt="maven6.png"></p>

这个文件中，我们需要注意的有：
modelVersion：一定是4.0.0，不管你是maven2,3,4，这个模型版本都是4
packaging：打包的目标形态，有war，jar，ejb等等
GAV坐标：这是唯一定位依赖的坐标体系。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven7.png" alt="maven7.png"></p>

GAV不光在项目定义中存在，在dependency中一样存在，它是整个maven依赖管理的核心概念。
groupId：一般就是组织名称，包名，比如 org.springframework
artifactId：项目名称，比如 spring-core
version：版本号，这里支持多种版本定义，比如RELEASE和SNAPSHOT，如果是定义了-SNAPSHOT版本，每次构建的时候，都会从仓库（非本地）拉取最新的版本，如果是RELEASE版本（或者没有指定SNAPSHOT），会从本地仓库拉取，如果要更新，必须更改版本号才会拉取新的RELEASE版本依赖（当然，你可以删除本地仓库指定的目录，以迫使maven再次拉取这个依赖）。
在dependency中，除了GAV，有时候我们还会用到 scope 选项，这个选项指定了依赖存在时期，支持：compile，runtime，provided，test，system。
简单说明一下：compile就是编译，测试，运行期间会用到的依赖，会打包到目标文件中，是默认的scope（可以不写）；runtime是运行期间会用到的依赖，不参与编译，但需要打包到目标文件；provided是外部会提供的依赖，但编译的时候会用到，会在编译期间拉取依赖，但打包不会打包到目标文件中；test是仅仅是单元测试的时候会用到（如junit.jar等），正式环境及打包都不会加入；system与provided一样，只是不从仓库拉取，而直接使用本地的jar。

plugins选项，maven是微服务架构，整个项目很小（解压后9MB），下载下来的maven仅仅包含必要的内核和cli工具，剩下的所有工具（命令）都是作为插件下载的，比如你第一次输入命令

~~~bash
mvn idea:idea
~~~

就会看到cli开始downloding一堆相关的jar包了。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven8.png" alt="maven8.png"></p>

maven内置了很多插件，那maven是如何将命令映射到pom.xml配置的呢？有兴趣可以查看localRepository目录中org/apache/maven/plugins 子目录中所有的插件明细（maven本身的依赖及工具，也是使用同一套体系建立起来的--自举能力）具体每个插件绑定到哪个阶段，以及使用的别名，在pom.xml都有定义（这里给出一个compile插件的pom点击查看 ），但细节部分，涉及到maven组件开发，有兴趣可以参考https://yq.aliyun.com/articles/488086 。

### Maven中的生命周期

因为Maven定位是项目管理工具，并将参与项目构建全过程，包括编译，测试（单元测试），打包，发布等过程。所以Maven将项目的生命周期做了抽象，目前有3种生命周期定义：clean、default、site。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven9.png" alt="maven9.png"></p>

- clean主要用于清理编译期生成的临时文件；    
- default是编译、测试、打包的主要过程；    
- site是生成项目相关文档，测试报告的过程。    

上面的default部分，仅列出主要阶段，如果详细列出的话，是有23个阶段的。每个周期都是依据顺序，从上至下依次执行的，如果某个阶段出现错误，整个构建过程就会终止，仔细观察顺序，其实就是我们平时手工构建Java代码的顺序，先验证，然后编译，如果编译没有问题，再进行单元测试，如果单元测试都通过了，才进行打包，这中间任何步骤出错，均需要停止构建工作。
当然，如果需要skip某个过程，可以通过命令或者pom的设置来作特殊处理。

我们常见的命令：
~~~bash
mvn clean package -DskipTests=true
~~~

就是按顺序使用了clean命令，清理上次编译期生成的临时文件，package命令绑定的是default周期的package阶段，会顺序执行validate,comiple,test,和package的任务，由于这个命令加入了-DskipTest=true参数，所以构建过程中会跳过test步骤。
以上cli命令都对应了一堆内置的核心插件。对于第三方的插件，我们可以在pom.xml中加入相应的plugin，然后直接执行，上面举例的idea:idea就是一个三方的插件，它将当前maven工程转为idea项目结构。
下图是springboot的插件，可以直接使用springboot:run任务来启动该springboot项目。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven10.png" alt="maven10.png"></p>

在idea等主流IDE中都提供的maven插件，可以直接给出执行goal的菜单，双击即可执行该maven的task或goal。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven11.png" alt="maven11.png"></p>

这里给出一个建议，如果是web项目，建议在plugins中加入jetty（嵌入式容器引擎）插件，这样可以快速启动web项目而不需要将war部署到tomcat或者其他容器，在开发测试阶段比较方便。

关于生命周期，可以参考文章http://www.cnblogs.com/davidwang456/p/3915031.html

### Maven版本处理

最后一部分，简单说一下依赖的版本处理方式，maven提供的版本依赖处理主要有两种

- 路径最短原则      
当A包依赖B包，B包依赖C包（C1），同时项目中D包依赖C包的另外一个版本（C2）。
这个时候会采用路径最短规则。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven12.png" alt="maven12.png"></p>
如上图所示，根据最短路径原则，最后Libs包含的jar包只会有C(2)版本的jar

- 声明优先原则    
当两条依赖路径一样，如何处理呢？
如下图

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven13.png" alt="maven13.png"></p>

这个取决于你在pom.xml中dependencies中各个依赖的定义顺序。

> 提问：maven如何知道各个依赖的二级依赖的版本？

查看各个依赖的pom.xml就可以分析出二级依赖，然后再通过二级依赖的GAV再从仓库下载二级依赖，然后根据二级依赖的pom.xml获取三级依赖的信息，如此递归，直到最后所有依赖分析完毕，然后maven会处理相应的依赖。所以得出结论，在项目中，maven会把所有的依赖jar和pom.xml都更新到本地仓库，然后再分析及处理依赖间的版本问题，最后打包的时候，会分拣出必要的jar。

### 其他

- 关于私服

可以使用nexus快速搭建一个maven私服，然后将该服务器地址配置到mirrors里面即可；
可以参考https://yq.aliyun.com/articles/7427我们公司的私有仓库也是采用nexus部署的。

- 国内的镜像库

推荐阿里云
http://maven.aliyun.com/mvn/view 

~~~xml
<mirror>
  <id>nexus-aliyun</id>
  <mirrorOf>*</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
~~~

- 如何将已有的历史项目移植为maven项目

如果之前采用的Ant或者其他构建方式，建议先梳理好Libs中的依赖关系，并从https://mvnrepository.com/中分别找到他们，这里仅关心一级依赖，将二级依赖的jar先去除，让maven自己从依赖自身的依赖定义来管理二级，三级...的依赖包。
如果发现私有jar或者仓库中没有的jar，可以使用

~~~bash
mvn install:install-file
~~~

命令来将本地的jar按照maven的方式（GAV）安装到本地仓库，然后就可以在pom.xml中使用了。
install-file与标准提交到仓库的依赖有什么不同呢？虽然都是jar，但由于install-file提交的依赖，其pom.xml只有自身（jar）的GAV信息，其dependencies是空的，所以，它的依赖（二级...依赖）你还得自己处理，这就是私有包与maven管理包的最大区别。
下图是我们内部系统的一个私有jar包安装到本地仓库的脚本

<p style="text-align: center;"><img src="http://store.tufeiping.cn/maven14.png" alt="maven14.png"></p>

- Gradle号称Maven终结者，它们的区别在哪里？

Gradle采用groovy语言（基于JVM的脚本语言）作为配置领域语言（采用groovy语言特性，使其看起来与配置文件无异），非常简洁高效。由于配置文件采用脚本语言编写，所以，除了能够覆盖到maven定义的要素，还能做原来Ant和Shell脚本做的事情，无疑，其扩展能力从简单程度上就大大优于maven了（maven需要开发插件）。
但在其他方面，特别是思想上，两者是相似的。各位掌握maven了，再使用Gradle一定会得心应手，如鱼得水。