### 安装Python

略

### 安装Flask

	pip install Flask

搭建虚拟环境推荐使用(`pipenv`)[https://docs.pipenv.org]

其余略

### 一个"Hello, World" Flask应用

(Flask官网)[http://flask.pocoo.org] 用了一个只有5行的简单应用例子来欢迎您的到来。所以在这里就不重复了，我将介绍一个稍微详尽的例子，这个例子基础结构比较好有利于编写更加复杂应用。

这个应用存在于一个包里，在python中，如果一个目录中包含一个`__init__.py`文件，那么这个目录就被认为是一个个包。当你导入一个包时，`__init__.py`文件会自动运行并且定义出这个包对外公开的变量或函数。

让我们建立一个包叫“app”，里面存放应用代码。然后在microblog目录下运行以下代码：
	
	(venv) $ mkdir app

`__init__.py`里面写入以下代码：

~~~python
from flask import Flask

app = Flask(__name__)

from app import routes
~~~

以上代码创建了一个Flask的实例，`__name__`变量代表当前模块名。如果需要导入模板（第二章会具体展开）等相关资源的时候，Flask会把当前模块位置作为起始位置。实际上，传递`__name__`变量几乎总是正确的方式配置Flask。然后应用会导入routes模块。

起初有一点可能会引起疑惑，就是存在2个app变量。实际上`from app import routes`中的app是指app目录，而另一个app是指我们创建的Flask实例。

另一个不同寻常的地方是我们在文件底部导入了routes模块，而不是像往常一样在顶部。那是因为底部导入是一个对循环导入的解决方法，这是Flask应用程序的常见问题。你待会就会看到routes模块需要导入`__init__`中定义的app变量，所以把它放在底部可以规避互相应用导致的问题。

所以routes模块里是什么呢？路由简单来说是指应用对各种不同URLs的实现方法。在Flask中用来处理各种不同路由被写成了python函数，叫做视图函数。视频函数被映射到一个或多个路由，所以当客户端发送一个请求时Flask应用知道该用什么样的逻辑去执行。

下面是你的第一个视图函数，你需要把他放在routes.py文件里：

~~~python
from app import app

@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"
~~~

这个视图函数其实非常的简单，它返回以一个字符串形式的问候。2个在函数上的`@app.route`实际上是装饰器。装饰器用来修改一个紧挨着它的一个函数的行为，它最常用的就是给特定的事件注册回调函数。在这个例子里，装饰器在给定的URls和函数之间建立起了联系，意思就是说当浏览器发起这2个URLs请求的时候，Flask就会调用他们对应的函数，然后返回对应内容给浏览器。如果你现在不太理解的话，你运行一下应用就会比较直观了。

为了完成整个应用，你需要在应用顶级目录下创建一个python脚本，在里面定义Flask实例。我这里把它命名为microblog.py：

~~~python
from app import app
~~~

这2个不同的app含义不同，上面已经介绍过就不累述了。

然后现在的整个应用目录应该是这样：

~~~
microblog/
  venv/
  app/
    __init__.py
    routes.py
  microblog.py
~~~

这样第一版的应用就完成了。在运行之前先设置应用坏境变量：

	$ export FLASK_APP=microblog.py

如果是windows系统的话把`export`改成`set`。现在你可以开始运行了：

~~~
$ flask run
 * Serving Flask app "microblog"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
~~~

服务器初始化结束之后就会进行监听，等待客户端请求。`flask run`的输出信息表示服务器的IP是127.0.0.1，也就是你自己计算机的地址，一般也可以叫做localhost。开发模式的监听的端口在：5000，应用正式上线后端口一般为443或80。现在你打开一下地址就可以看到运行的结果：

	http://localhost:5000/

或者

	http://localhost:5000/index

!()[https://blog.miguelgrinberg.com/static/images/mega-tutorial/ch01-hello-world.png] 

因为装饰器给2个不同的地址绑定了相同的视图函数，所以返回值是一样的。

当你想退出服务器监听的时候可以按下Ctrl-C。

恭喜你，你已经成功完成了通向web开发者的第一步了。

