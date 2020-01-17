## Python的Decorator

> 原创 涂飞平  `2018-10-23`

<p style="text-align: center;"><img src="http://store.tufeiping.cn/pythodecorator.png" alt="pythodecorator.png"></p>

`Python`语言中的`Decorator`，也就是常说的装饰器，个人认为是`Python`语言中的亮点之一，在语言层面（从语义上）直接支持装饰器的我所知道的也就是`Python`了，虽然`JavaScript`也是支持的，但需要通过技术手段来处理一下（都是将函数赋值给同名的变量以达到相同的效果），而`Python`虽然在原理上是一致的，但其在语言层面直接支持，提供了 `@` 运算符来标识（跟`Java`提供的注解采用一样的标识符，而且作用也类似），更直观，更简洁！

对于`Decorator`的作用，可以简单概括为：**函数对象的偷梁换柱！**

就是将一个新函数替换原来的函数，由于`Python`和`JavaScript`中的函数都是对象（*First Class Object*），所以可以赋值给指定的变量，这个变量名 就是我们所说的函数名。这有什么用处呢？



1. 函数补丁，即修改原有函数的*bug*，在作为框架或者产品代码的库中，也许已经大量调用了某个函数，但该函数有问题，这个时候，如果能重新修改该函数，且不影响原有代码，`Decorator`就可以解决该问题；

2. 扩展函数功能，可以扩展原有函数的功能（熟悉`Spring`的就能知道，这其实就是**AOP**的思想），如果你想给你已经写好的函数添加一些新的功能(比如调用日志) 你可以使用新的函数替换原有函数（当然，在新的函数中还是要调用原有的函数），然后将该新函数替换原有函数就可以解决这个问题，`Decorator`也是完美的解决方案。



在`JavaScript`中，我们需要扩展一个名为`test`的函数，比如

~~~javascript
var test = function() {
    console.log("hello world");
}
~~~

给它加上日志功能，代码如下

~~~javascript
decorator = function(fn/*你看，这里函数就是作为参数进行传递*/) {
    return function(){
        console.log("before test");
        fn();
        console.log("after test");
    }
}
~~~

然后，替换的时候，只要完成一次调用即可： 

~~~javascript
test = decorator(test);
~~~

其中的`fn`就是传入的函数对象，在返回的新的函数对象中，`fn`会保存在作用域链中，（闭包的原理，同样适用于`Python`）

而调用的时候，完全可以不用去理会其中函数的细节是否发生变化

~~~javascript
//...其他代码 
test(); 
//...其他代码
~~~

在`Python`中，也可以举出一模一样的例子来：

~~~python
def test():
    print "hello world"
	
~~~
	
也给他加上日志功能，代码如下：

~~~python
def decorator(fn):
    def newTest():
        print "before"
        fn()
        print "after"
    return newTest
~~~

类似的替换方式：

~~~python
test = decorator(test)
~~~

然后，也是同样的调用方式： 

~~~python
#...其他代码 
test() 
#...其他代码
~~~

在`Python`中，可以用语法糖 `@` 来简化代码，使得代码更简单优雅，比如上面的代码： 

~~~python
test = decorator(test)
~~~

可以参考一下，下面的代码其实就等效于上面的代码，看起来是不是简洁很多了？ 

~~~python
@decorator 
def test():
~~~

了解原理之后，就可以理解`Python`库中的各种装饰器调用，概括起来：使用新的函数对象来包装旧的函数对象，以扩展或者替换原有对象。

下面简单分析几种`Python`中的装饰器使用方式：

1. 简单的装饰器，没有参数，只是做简单的工作，如上面的例子所示；

2. 带有参数的装饰器，带有参数，根据参数来修改工作函数的行为，比如`Bottle`中使用`Route`装饰器来分发访问请求(以下代码摘自http://sunnytu.sinaapp.com的，该应用采用`Bottle` /* 备注：这是很早以前，我在新浪云部署的应用，采用的是Bottle Web框架，目前已经废弃了 */)

~~~python
@app.route('/html/:fn') 
def sundy_static(fn): 
    return static_file(fn, root="static")
~~~

过多的分析，可能会把人搞晕，我还是直接写出`route`的示意代码吧：

~~~python
def route(url_pattern):
    def newroute(fn):
        def newproc():
            #process with url_pattern, get params
            params = xxx
            return fn(params)
        return newproc
    return newroute
~~~

而@route('/html/:fn')所做的工作如下： 

~~~python
sundy_static = route('/html/:fn')(sundy_static)
~~~

`Bottle`会将原始的 *sundy_static* 加入路由列表，以便在处理请求的时候找到所需的处理例程，加入参数后的装饰器更加强大，与**AOP**表达式类似，可以进行逻辑处理（如何替换旧的函数对象，或者生产新的对象）；

3. 对象方法的装饰器，其实与一般方法类似，只是要注意特殊参数`self`，比如：

~~~python
class test(): 
    def sayHello(self): 
        print "Hello world"


def decorator(fn): 
    def newHello(self): 
        print "before" 
        fn(self) 
    return newHello
~~~

如何使用呢？ 直接在类定义中，方法前面加入该装饰器即可： 

~~~python
class test(): 
    @decorator 
    def sayHello(self):
~~~

`Python`有一些比较常用的`Decorator`，比如`staticmethod`, `classmethod`等 关于闭包，你可以参考这个链接（http://developer.51cto.com/art/201006/208139.htm）。

`Decorator`应用模式，其实就是代理模式，也是实现**AOP**（面向切/方面编程）的基础。

`JavScript`和`Python`是原生支持**AOP**的，`Java`和`C#`可以通过动态代理或者第三方工具可以达到相同的效果。