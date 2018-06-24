在第二章中我创建了一个简单的主页模板，然后用虚拟的用户和博客文章暂时用来完成功能。在这一章中，我们将填补一个坑，就是怎样从表单中获取用户的输入。

web表单是任何web应用中都比较基础的一块内容。我们将用表单允许用户提交日志等内容，当然也可以用来提交登陆。在你开始学习本章内容之前，请确保已经完成之前那两章的内容。

### Flask-WTF简介

我们将使用Flask-WTF扩展来处理web表单。Flask-WTF是[WTForms](https://wtforms.readthedocs.io/)包在Flask下的一个整合扩展。这是我在本教程里介绍的第一个扩展，但不会是最后一个。其实扩展是FLask生态系统中非常重要的一部分，由于Flask自身的简介设计所以很多问题并没有整合解决方法，而这些扩展起到了巨大的作用。

Flask扩展也是寻常的python包，你可以直接用pip来安装：

	$ pip install flask-wtf

### 配置

到目前为止应用还是非常的简单的，所以并没有涉及到应用配置的问题。但是除了那些非常简单的应用，大部分的应用和扩展都给了你非常大的自由可以自定义他们的行为，而你只需要简单给框架传递一组变量就可以了。

具体有一下几种方法可以用来配置选项。最基础的方法就是直接定义在`app.config`这个字典中。例如：

~~~python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'you-will-never-guess'
# ... add more variables here as needed
~~~

尽管如此设置比较简洁有效，但是我还是建议分而治之，把设置部分单独写进一个文件。把配置参数写进一个python的类，这样条理清晰又容易扩展：

~~~python
config.py: 

import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'you-will-never-guess'
~~~

`SECRET_KEY`参数是大部分应用都非常重要的。因为Flask和部分扩展用它来生成就签名或者令牌，Flask_WTF扩展用它来保护表单不受[CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery)侵害。有兴趣可自行深入了解。

`os.environ.get('SECRET_KEY') or 'you-will-never-guess'`意思是优先用环境设置的安全值，仅当把应用部署到服务器上时。在开发阶段只需使用自己生成的安全值即可。

现在我们有了一个配置文件，那么如何导入呢：

~~~python
from flask import Flask
from config import Conifg

app = Flask(__name__)
app.config.from_object(Conifg)

from app import routes
~~~

### 用户登录表单

Flask-WTF扩展用python类来代表web表单，表单中的各个框用类变量来定义，比如用户名框、密码框等等。

一贯的分而治之原则我们给表单单独创建一个文件`forms.py`。先来定义一个用户登录表单，包含一个用户名框和密码框，还有一个“记住我”的复选框和一个提交按钮：

~~~python
from flask_wtf import FlaskForm
from wtforms import StringField,PasswordField,BooleanField,SubmitField
from wtforms.validators import DataREquired

class LoginForm(FlaskForm):
	username = StringField('Username',validators=[DataRequired])
	password = PasswordField('Password',validators=[DataRequired])
	remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')
~~~

大多数的Flask扩展名顶层导入的标示符都遵从`flask_<name>`的习惯。在这里，Flask-WTF的标示符都在`flask_wtf`里，`FlaskForm`基础类就是从这里导入的。

另外4个代表输入框的类都是直接从WTForms包里导入的，因为Flask-WTF扩展不提供定制的输入框。在`LoginForm`类里面，每一个输入框都创建一个类变量，这个类的第一个参数是这个框的描述或者标签。

可选的参数`validators `是用来自定义输入框的行为的，`DataRequired`验证器是用来验证这个输入框是否提交了空值。另外还有很多可用的验证器。

### 表单模板

下一步是完成表单模板部分：

~~~html
{% extends "base.html" %}

{% block content %}
	<h1>Sign In</h1>
	<form action="" method="post" novalidate>
		{{ form.hidden_tag() }}
		<p>
			{{ form.username.lable }}<br>
			{{ form.username(size=32) }}
		</p>
		<p>
			{{ form.password.label }}<br>
            {{ form.password(size=32) }}
		</p>
		<p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock%}
~~~

稍微解释一下。这个模板文件base.html还是第二章中说过的那个模板，它需要传入一个LoginForm的实例form，稍后的视图函数会来完成这个操作。

`<form>`元素在这里作为一个容器，`action`属性用来告诉浏览器当表单提交的时候用什么URL来处理，如何设置为空值意思就是一当前的URL来处理提交。`method`属性标明了这个表单提交时所使用的HTTP请求方法，默认的方法是`get`，但是大部分情况下提交表单最好使用`post`请求方法。因为`get`方法提产能会把输入的参数加在URL中，而`post`会把提交的输入放在请求的body中。关于HTTP协议如有兴趣可自行搜索教程。

`novalidate`属性告诉浏览器不要验证输入的值，这样的话就把这个验证的任务留给了Flask应用的服务器端。第一次最好这样设置，因为后续我们要测试一下服务器端的验证是否有效果。

`form.hidden_tag()`是用来预防CSRF攻击的，有兴趣课自行搜索。简单来说就是为了验证此次提交是正对当前网页下的。试想一个案例：当一个网页被做了手脚，你以为是在登录一个网上银行，其实这个form确是给另外一个账户转账，你提交的同样的是账号密码的情况下你的钱就在不知不觉中被转走了。`form.hidden_tag()`会使用上文中提及的安全值生成一个令牌，确保你的提交是只针对当前网页的提交。

你只需完成以上，剩下的事情模板引擎会替你渲染的。

### 表单视图函数

最后的一步就是编写视图函数了：

~~~python
from flask import render_template
from app import app
from app.forms import LoginForm


@app.route('/login')
def login():
    form = LoginForm()
    return render_template('login.html', title='Sign In', form=form)
~~~

导入`LoginForm`类创建实例，并把实例传递给模板引擎渲染。

为了更加方便的进入登录页面，可以在模板中添加导航栏：

~~~html
<div>
    Microblog:
    <a href="/index">Home</a>
    <a href="/login">Login</a>
</div>
~~~

现在你可以运行一下应用试试：

![](https://blog.miguelgrinberg.com/static/images/mega-tutorial/ch03-login-form.png)

### 获取表单数据

到目前为止视图函数还只能渲染登录模板，还不能处理用户提交的数据，我们现在来更新一下视图函数：

~~~python
from flask import render_template, flash, redirect

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
~~~

`methods`参数告诉Flask这个视图函数接受的HTTP请求类型，默认参数只有get。`form.validate_on_submit()`方法会验证所有用户提交的信息，如果符合条件则返回True。接着`flash()`函数负责显示出一行信息提示。`redirect()`函数用来跳转到其他URL。

当你使用`flash()`函数时，Flask会先储存信息，信息不会自己显示在网页上。模板必须进行渲染才能显示出来：

~~~html
base.html:
<html>
    <head>
        {% if title %}
        <title>{{ title }} - microblog</title>
        {% else %}
        <title>microblog</title>
        {% endif %}
    </head>
    <body>
        <div>
            Microblog:
            <a href="/index">Home</a>
            <a href="/login">Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>
~~~

`get_flashed_messages()`是Flask框架自带的，它返回所有`flash()`函数产生的信息。这些信息显示一次后就会被删除，所以每条信息只会显示一次。现在你可以再尝试运行一下应用。

### 加强表单验证

到目前为止用户如果输入了不符合条件的信息，服务器端验证以后只会再次返回表单让用户重新填写。现在我们需要提升用户体验，在用户输入的时候就提示相关的错误信息：

~~~html
{% extends "base.html" %}

{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post">
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}<br>
            {% for error in form.username.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}<br>
            {% for error in form.password.errors %}
            <span style="color: red;">[{{ error }}]</span>
            {% endfor %}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
~~~

其实这些错误信息在验证器里都已经写好了，你只需在模板中添加进去即可。按照惯例，任何添加了验证器的表单都会把错误放在`form.<field_name>.errors`中。如果你试图提交空值的话就会有以下提示：

![](https://blog.miguelgrinberg.com/static/images/mega-tutorial/ch03-validation.png)

### 生成链接

到目前为止用户登录表单就算告一段落了。现在要说一下生成链接的最佳实践。我们之前在模板中直接把链接地址写死了：

~~~html
<div>
    Microblog:
    <a href="/index">Home</a>
    <a href="/login">Login</a>
</div>
~~~

但万一你想要重新组织你的链接，那就不得不一个个去修改。Flask提供了一个极好的函数：`url_for()`，它利用框架内部URLs到视图函数映射关系产生链接。例如`url_for('login')`返回`/login`等：

~~~html
<div>
    Microblog:
    <a href="{{ url_for('index') }}">Home</a>
    <a href="{{ url_for('login') }}">Login</a>
</div>
~~~

~~~python
from flask import render_template, flash, redirect, url_for

# ...

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for('index'))
    # ...
~~~


