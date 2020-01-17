## git https/http方式下免密登录

> 原创 涂飞平  `2018-10-16`

<p style="text-align: center;"><img src="http://store.tufeiping.cn/git.png" alt="git.png"></p>

在使用Git的时候，一般情况下都是使用http/https方式，这个种方式虽然比ssh免密登录多了一个输入用户名/密码的步骤，但由于其不需要将公钥存入git server，所以目前是我们部门的推荐（默认）使用方式。
在http/https模式下，每次进行pull，push（与服务器同步）操作，都需要输入用户名及密码，比较繁琐，这里总结了3种方式来避免http/https模式下每次输入账户信息的问题。

### URL带账户信息

在git做clone或者新增remote的情况下，将用户名和密码信息加入到url种，如下

~~~txt
git clone http://username:password@git.yonyou.com/shuiwu/taxrisk.git
~~~

或者

~~~txt
git remote add yonyou http://username:password@git.yonyou.com/shuiwu/taxrisk.git
~~~

这种方式需要注意：
- 如果账户名中包含@的邮箱格式，需要将@编码为%40（两个@会导致git提交校验失败），比如：xxx@gmail.com->xxx%40gmail.com    
- 将账户信息直接放在仓库的remote URL中，可以通过git remote -v 查看，不是太安全

### 本地保存账户信息

通过config设置将用户信息保存到本地。如下：
~~~txt
git config --global credential.helper store
~~~

这个方式是将用户登录信息保存在本地文件（相对方式1安全一点点）
~~~txt
$HOME/.git-credentials
~~~

文件中，同时会在.gitconfig文件中添加如下一行信息：

~~~txt
[credential]
	helper = store
~~~

如果有多个账户信息，可以直接编辑.git-credentials，添加多行记录即可

### _netrc文件

这是比较推荐的方式，实现方式与方式2类似，也是采用本地文件来存储登录信息

~~~txt
$HOME/_netrc
~~~

文件格式如下：

~~~txt
machine git.yonyou.com
login yyyyy
password xxxxx
~~~

这个方式是我比较推荐的，如果有多个仓库，依次设置即可。
通过以上任何一种方式设置后，再执行git push或者git pull的时候，都不再需要输入用户账户信息了，比较方便。

### 参考资源


1. https://www.cnblogs.com/wish123/p/3937851.html
2. https://www.cnblogs.com/liuchao102/p/5499192.html