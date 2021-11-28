# flask
---

> Flask 中文文档（2.0.1）
Flask 以来 Jinja 模板引擎和 Werkzeug WSGI 套件

[原文档地址](https://dormousehole.readthedocs.io/en/latest/index.html)


## 一、安装

### python 版本
> Flask 支持 Python 3.6 及更高版本

### 依赖
> 当安装 Flask 时，以下套件软件会被自动安装
* Werkzeug 用于实现 WSGI，应用和服务之间的标准 Python 接口
* Jinja 用于渲染页面的模板语言
* MarkupSafe 与 Jinja 共用，防止注入攻击
* ItsDangerous 保证数据完整性，保护 Flask 的 session、cookie
* Click 是一个命令行应用框架。用于提供 Flask 命令，并允许添加自定义管理命令

### 可选依赖
* Blinker 为信号提供支持
* python-dotenv 当运行 flask 命令时通过 dotenv 设置环境变量提供支持
* Warchdog：为开发服务器提供快速高效的重载

### 虚拟环境
> 为每个项目安装单独的 Python 库，隔离不同项目间的 Python 库
Python 内置了用于创建虚拟环境的 venv 模块


### 创建虚拟环境
> 创建完成后文件夹中会生成一个 venv 文件夹

```shell
# 创建
python3 -m venv venv
# 激活
. venv/bin/activate
```



### 安装 Flask

```shell
pip install Flask
```

## 二、快速上手

### 一个最小的应用
```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello_world():
    return "<p>Hello, World!</p>"
```
1. 导入 Flask，该类的实例会成为 WSGI 应用
2. 使用 route() 指定 http url
3. 接口默认返回 html

`不要使用 flask.py 作为应用名称，会与 Flask 本身冲突`


运行方式：
1. hello.py  ->  export FLASK_APP=hello  ->  flask run --host=0.0.0.0
2. app.py(wsgi.py)  ->  flask run --host=0.0.0.0


### 路由
> 使用 route() 装饰器把函数绑定到 URL

```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'
```

### URL 中使用变量

```python
from markupsafe import escape

@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return f'User {escape(username)}'

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return f'Post {post_id}'

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return f'Subpath {escape(subpath)}'
```

### URL 构建
> url_for() 函数用于

### HTTP 方法
> Web 应用使用不同的 HTTP 方法处理 URL。缺省情况下，一个路由只回应 GET 请求

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

### 静态文件

创建 static 目录，将静态资源放在该目录中
```python
url_for('static', filename='style.css')
```

### 渲染模板
> 使用 render_template() 方法可以渲染模板，只需提供模板名称和作为参数的变量

```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

### 请求对象
> from flask import request

```python
@app.route('/login', methods=['POST', 'GET'])
def login():
    error = None
    if request.method == 'POST':
        if valid_login(request.form['username'],
                       request.form['password']):
            return log_the_user_in(request.form['username'])
        else:
            error = 'Invalid username/password'
    # the code below is executed if the request method
    # was GET or the credentials were invalid
    return render_template('login.html', error=error)
```

### 文件上传
> 表单中设置 enctype="multipart/form-data"

```python
from flask import request

@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
    ...

```

### cookie

读取 cookie
```python
from flask import request

@app.route('/')
def index():
    username = request.cookies.get('username')
    # use cookies.get(key) instead of cookies[key] to not get a
    # KeyError if the cookie is missing.
```

存储 cookie
```python
from flask import make_response

@app.route('/')
def index():
    resp = make_response(render_template(...))
    resp.set_cookie('username', 'the username')
    return resp
```

### 重定向和错误
> 使用 redirect() 函数可以重定向，使用 abort() 可以更早退出请求，并返回错误码

```python
from flask import abort, redirect, url_for

@app.route('/')
def index():
    return redirect(url_for('login'))

@app.route('/login')
def login():
    abort(401)
    this_is_never_executed()
```

自定义错误页面
```python
@app.errorhandler(404)
def page_not_found(error):
    return render_template('page_not_found.html'), 404
```

### 日志

```python
app.logger.debug('A value for debugging')
app.logger.warning('A warning occurred (%d apples)', 42)
app.logger.error('An error occurred')
```


...