## 如何在Docker中安装DzzOffice

> 原创 涂飞平  `2014-05-18`

### 一些背景

从2012年底开始听说并开始研究Docker了，并且在公司内部小规模使用了，当时主要的场景是将Docker作为内部的一个小的 `PAAS` 基础，提供一致，统一的编译/测试环境，分配给新来的开发人员使用。


使用Docker一段时间之后，准备自己倒腾使用Docker + Hadoop(HDFS) 组建一个私有存储云环境，统一管理自己的产品/项目内部的代码、文档。

选择先从文档管理开始，在底层架子搭建好之后，需要给内部人员提供一个好的“门面”，在没有这个门面之前，我们大量使用 `SSH+Command` 方式操作，这种方式对于公司内部非RD部门的人员，门槛还是很高的，他们是不会使用这么晦涩的东西的。

前几天，在OSC上面偶然发现了 **DzzOffice**，当时看到介绍后，直觉认为它是可以胜任目前的需求的。



1. 界面实在是漂亮；

2. 支持多种接口，阿里云、百度云和本地存储，为以后升级提供了可能。



为了更进一步了解DzzOffice，加入到DzzOffice QQ群（240726），知道DzzOffice对于HDFS的支持目前暂时还没有提供，但这应该不是问题，DzzOffice会不断更新并持续加入新的支持。



先在官网 http://www.dzzoffice.com/ 上面了解了一下DzzOffice依赖的环境和背景，它运行在典型的LAMP/WAMP平台上。是否可以在公司内部服务器Docker群中加入一个Docker容器来专门运行DzzOffice？经过实际测试，很轻松就完成这个设想，很完美。为了便于大家在Docker上面体验DzzOffice，我将Docker工程提交Github上面，接下来就根据Github上面的工程来一步一步安装DzzOffice。



### 安装步骤

我已经在Github上面创建了dzzwithdocker项目，可以到这里查看项目

https://github.com/sundytu/dzzwithdocker

国内gitee地址为

https://gitee.com/sunnytu/dzzwithdocker



1. 下载项目

Git的项目，你可以clone或者直接下载zip包到本地，然后解压到一个目录下面，这里假定你把系统解压到/home/cores/dockers/dzz1.0目录下面


2. 创建dzz镜像

进入到 `dzz1.0` 目录中，通过执行 

~~~bash
docker build -t dzz10 . 
~~~

生成dzz10镜像，由于国内网络情况，这是一个漫长的过程，我花费了大概半个小时（不包含ubuntu镜像的下载时间，因为之前已经下载完成该部分镜像）。可以改用国内的docker registry，会快很多！

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dockerdizz1.png" alt="dockerdizz1.png"></p>

如果安装失败，是网络原因，请重复执行几次，肯定能成功的。

然后执行 docker images 命令查看是否有dzz10名称的image是否创建成功。



3. 启动dzz容器

完成dzz1.0镜像的构建，通过 docker run命令启动一个容器，并将80和22端口映射到宿主机器端口上面，便于我们在浏览器中访问，我这里将80端口映射到8081端口，命令如下：

~~~bash
docker run -d -p 8081:80 -p 2221:22 dzz10
~~~


4. 配置DzzOffice

在浏览器中输入 http://127.0.0.1:80801 来访问系统，如果看到如图界面，恭喜您，安装DzzOffice成功了，剩下就是按照提示进行DzzOffice的配置了

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dockerdizz2.png" alt="dockerdizz2.png"></p>

经过一番配置，可以看到界面了，确实华丽，赏心悦目 :-)

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dockerdizz3.png" alt="dockerdizz3.png"></p>

### 单机安装

为了在Github上面创建dzzwithdocker项目，我在自己本子（Thinkpad T420i 8G内存）上使用Vageant + VirtualBox方式部署coreos系统，然后在coreos环境中下载dzzwithdocker项目文件，解压，然后构建docker镜像，运行容器，通过浏览器（Chrome）设置，访问，一切正常，下面是整个系统的图示。

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dockerdizz4.png" alt="dockerdizz4.png"></p>

还是华丽的界面 :-)

<p style="text-align: center;"><img src="http://store.tufeiping.cn/dockerdizz5.png" alt="dockerdizz5.png"></p>

向不断努力进取的 http://www.dzzoffice.com/ （社区版本免费，比较厚道）致敬！



涂飞平 `2014-05-18` 北京
 