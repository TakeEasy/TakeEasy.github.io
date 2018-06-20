---
title: 玩蛇笔记WEB篇之Tornado
date: 2017-07-06 19:57:06
categories:
- 开发
- Python
- WEB
tags:
- Python
- WEB
- 框架
- Tornado
---
{% cq %} 人生苦短,我玩Python {% endcq %}
{% fi /img/12.png, ..., Python %}
<!-- more -->
> Python WEB框架 先来Tornado吧...
> 2017-07-06:更新 概述,框架使用中的快速开始.

------

### 概述

Tornado 是 FriendFeed 使用的可扩展的非阻塞式 web 服务器及其相关工具的开源版本.这个 Web 框架看起来有些像web.py 或者 Google 的 webapp，不过为了能有效利用非阻塞式服务器环境，这个 Web 框架还包含了一些相关的有用工具 和优化.

Tornado 和现在的主流 Web 服务器框架(包括大多数 Python 的框架)有着明显的区别:它是非阻塞式服务器.而且速度相当快。得利于其 非阻塞的方式和对 epoll 的运用.Tornado 每秒可以处理数以千计的连接.这意味着对于实时 Web 服务来说，Tornado 是一个理想的 Web 框架.
{% codeblock 下载安装 %}
pip3 install tornado
或者用源码安装,官网下载tar包.
{% endcodeblock %}

------
### 框架使用
#### 快速开始

**下面所有的例子都基于此为基础,无非是添加Handler和路由索引.**
{% codeblock lang:python start_simple.py %}
{% raw %}
# -*- coding:utf-8 -*-
# Author:YEAR
import tornado.ioloop
import tornado.web

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello,world")

application=tornado.web.Application([
    (r"/index",MainHandler)
])

if __name__=="__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
{% endraw %}
{% endcodeblock %}
整个过程:
1. 执行py,监听本地8888端口
2. 浏览器客户端访问本地8888端口/index目录--http://127.0.0.1:8888/index
3. 服务器接收请求,有application里的正则表达式解析地址,选择对应对的Handler类操作.
4. 类接收到请求后,根据请求方法(post/get/delete...)执行对应的函数.
5. 方法返回的值返回给请求客户端上

#### 路由系统

所谓路由系统,就是一种寻址方式,就是根据用户请求的网址,来知道用户想要做什么或者需要得到什么样的信息.
{% codeblock lang:python route.py %}
application = tornado.web.Application([
    (r"/index", MainHandler),
    (r"/demo", demoHandler),
    (r"/login", loginHandler),
    (r"/logout", logoutHandler),
    (r"/publish", publishHandler),
    (r"/page/(?P<page>\d*)", pageHandler),  # 正则表达式索引
    (r"/extendInclude", eandiHandler),
    (r"/cookie", cookieHandler),
    (r"/cookielogin", cookieloginHandler),
    (r"/session", sessionHandler),
    (r"/sessionlogin", sessionloginHandler),
    (r"/checkcode", checkcodeHandler),
    (r"/checkcodeLogin", checkcodeLoginHandler),  # 带验证码登录的入口
    (r"/csrf", csrfHandler),
    (r"/ajax", ajaxHandler),
    (r"/fileupload",fileuploadHandler),
    (r"/corsAjax",corsAJAXHandler),
    (r"/checkform",checkFormHandler),
], **settings)

application.add_handlers('biubiu.year.cool$', [
    (r'/index', demoHandler)
])
{% endcodeblock %}
我们在创建tornado的application对象的时候就需要给定一个路由系统的对应列表,列表元素是一个包含地址内容和Handler类的元组.
Tornado中原生支持二级域名的路由.

#### 模板引擎
Tornado的模板引擎和Django中类似,模板引擎将模板文件载入内存,然后根据模板引擎中的模板语言和加载时传入的数据生成最终的字符串返回给请求用户,实际上跟大多数的模板引擎类似,只是模板语法不同.像JSP,ASP原理基本一样.
Tornado的模板语言主要有**控制语句**和**表达语句**,控制语句使用`{% raw %}{%{% endraw %}`和`{% raw %}%}{% endraw %}`包起来类似`{% raw %}{% if len(items) > 2 %}{% endraw %}`.表达语句是使用`{% raw %}{{{% endraw %}` 和 `{% raw %}}}{% endraw %}`包起来类似`{% raw %}{{ items[0] }}{% endraw %}`
控制语句和对应的Python语句的格式几乎完全相同,支持`for`,`while`,`if`,`try`只不过在逻辑结束的时候要加上`{% raw %}{% end %}{% endraw %}`,还通过`extends`和`block`实现模板继承.
在使用模板引擎前需要在setting中设置模板文件的路径.
1. 基本  
    {% codeblock lang:python app.py %}
    {% raw %}
    class MainHandler(tornado.web.RequestHandler):
        def get(self, id1, id2):
            # self.write("Hello,world")
            name = self.get_argument("yoyo", None)
            if name:
                INPUTS_LIST.append(name)
            self.render('index.html', yoyo=INPUTS_LIST, nmb='biubiubiu')

        def post(self, *args, **kwargs):
            print("post")
            name = self.get_argument("yoyo")
            print(name)
            INPUTS_LIST.append(name)
            self.render('index.html', yoyo=INPUTS_LIST, nmb='biubiubiu')
    {% endraw %}
    {% endcodeblock %}
    {% codeblock lang:html index.html %}
    {% raw %}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Tornado demo</title>
    </head>
    <body>
    <script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
    Hello,World!
        <form method="post" action="/index">
            <input type="text" name="yoyo"/>
            <input type="submit" value="提交" />
        </form>
        <h1>内容展示</h1>
        <h3>{{ func(nmb) }}</h3>
        <h3>{% module custom() %}</h3>
        <ul>
            {% for item in yoyo %}
                {% if item=='baba' %}
                    <li style="color:red">{{item}}</li>
                {% else %}
                    <li>{{item}}</li>
                {% end %}
            {% end %}
        </ul>
    </body>
    </html>
    {% endraw %}
    {% endcodeblock %}
    {% codeblock 默提供的方法 %}
    {% raw %}
    escape: tornado.escape.xhtml_escape 的別名
    xhtml_escape: tornado.escape.xhtml_escape 的別名
    url_escape: tornado.escape.url_escape 的別名
    json_encode: tornado.escape.json_encode 的別名
    squeeze: tornado.escape.squeeze 的別名
    linkify: tornado.escape.linkify 的別名
    datetime: Python 的 datetime 模组
    handler: 当前的 RequestHandler 对象
    request: handler.request 的別名
    current_user: handler.current_user 的別名
    locale: handler.locale 的別名
    _: handler.locale.translate 的別名
    static_url: for handler.static_url 的別名
    xsrf_form_html: handler.xsrf_form_html 的別名
    {% endraw %}
    {% endcodeblock %}

2. 使用母版文件的模板,以及导入别的文件内容的模板.
    {% codeblock lang:html layout.html %}
    {% raw %}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <link href='{{ static_url("bootstrap/css/bootstrap.css") }}' rel="stylesheet">
        {% block css %}{% end %}
    </head>
    <body>
        {% block body %}{% end %}
        <script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
        <script src='{{ static_url("bootstrap/js/bootstrap.js") }}'></script>
        {% block js %}{% end %}
    </body>
    </html>
    {% endraw %}
    {% endcodeblock %}
    {% codeblock lang:html form.html %}
    {% raw %}
    <form action="/">
        <input type="text" />
        <input type="submit" />
    </form>
    {% endcodeblock %}
    {% codeblock lang:html eandi.html %}
    {% extends './master/layout.html'  %}
    {% block css %}{% end %}
    {% block body %}
            {% include './content/form.html' %}{% end %}
    {% end %}
    {% block js %}{% end %}
    {% endraw %}
    {% endcodeblock %}

3. 自定义UIMethod和UIIModule
    
    1. 首先要定义它们

        {% codeblock lang:python uimethods.py %}
        {% raw %}
        # -*- coding:utf-8 -*-
        # Author:YEAR

        def func(self,arg):
        return "Hi"+arg
        {% endcodeblock %}
        {% codeblock lang:python uimodule.py %}
        # -*- coding:utf-8 -*-
        # Author:YEAR
        from tornado.web import UIModule

        class custom(UIModule):
            def render(self, *args, **kwargs):
                return '123'
        {% endraw %}
        {% endcodeblock %}

    2. 要在settings中对他们进行注册

        {% codeblock lang:python %}
        {% raw %}
        settings = {
            "template_path": "Template",  # 模板路径的配置
            "static_path": "Static",  # 必须配置,否者静态文件永远找不到
            "static_url_prefix": "/biubiu/",  # 静态文件路径前缀
            "ui_methods": md,  # 配置ui method.
            "ui_modules": mt,  # 配置ui module.
            "cookie_secret": 'asdfasdfasdfasdfasdf',  # cookie加密
            "xsrf_cookies": True  # 跨站请求伪造防止
        }
        {% endraw %}
        {% endcodeblock %}

    3. 在模板文件中使用它们

        {% codeblock lang:html %}
        {% raw %}
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>Tornado demo</title>
        </head>
        <body>
        <script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
        Hello,World!
            <form method="post" action="/index">
                <input type="text" name="yoyo"/>
                <input type="submit" value="提交" />
            </form>
            <h1>内容展示</h1>
            <h3>{{ func(nmb) }}</h3>
            <h3>{% module custom() %}</h3>
            <ul>
                {% for item in yoyo %}
                    {% if item=='baba' %}
                        <li style="color:red">{{item}}</li>
                    {% else %}
                        <li>{{item}}</li>
                    {% end %}
                {% end %}
            </ul>
        </body>
        </html>
        {% endraw %}
        {% endcodeblock %}

#### 静态文件
对于静态文件的使用,主要是css,js,页面资源等等的使用,可以配置静态文件的系统目录和使用时的前缀,tornado支持静态文件的缓存
{% codeblock lang:python settings.py %}
{% raw %}
settings = {
    "template_path": "Template",  # 模板路径的配置
    "static_path": "Static",  # 必须配置,否者静态文件永远找不到
    "static_url_prefix": "/biubiu/",  # 静态文件路径前缀
    "ui_methods": md,  # 配置ui method.
    "ui_modules": mt,  # 配置ui module.
    "cookie_secret": 'asdfasdfasdfasdfasdf',  # cookie加密
    "xsrf_cookies": True  # 跨站请求伪造防止
}
{% endraw %}
{% endcodeblock %}

{% codeblock lang:html index.html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tornado demo</title>
</head>
<body>
<script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
Hello,World!
    <form method="post" action="/index">
        <input type="text" name="yoyo"/>
        <input type="submit" value="提交" />
    </form>
    <h1>内容展示</h1>
    <h3>{{ func(nmb) }}</h3>
    <h3>{% module custom() %}</h3>
    <ul>
        {% for item in yoyo %}
            {% if item=='baba' %}
                <li style="color:red">{{item}}</li>
            {% else %}
                <li>{{item}}</li>
            {% end %}
        {% end %}
    </ul>
</body>
</html>
{% endraw %}
{% endcodeblock %}
静态文件的缓存实现 主流做法就是在静态文件的后面加`?v=版本号`版本号通常是文件的md5校验码的哈希之后的字符串.这样浏览器就可以每次都判断这个静态文件需不需要重新请求

#### Cookie
Tornado中可以对cookie进行操作,还可以对cookie进行签名防止伪造
要使用签名cookie,可以在settings里配置`cookie_secret`,然后使用`set_secure_cookie`和`get_secure_cookie`去设置和获取签名cookie.
关于签名cookie的原理, 其实就是将加密值和原本值加加密key加时间戳的加密值和时间戳拼在一起,服务器端在拿到之后,可以通过解码加密值得到值,再将值和时间戳和本地的加密key拼在一起加密 后与客户端的加密串比较即可.
{% codeblock lang:python cookie.py %}
{% raw %}
# cookie操作
class cookieHandler(tornado.web.RequestHandler):
    def get(self):
        print(self.cookies)  # 所有cookie
        self.set_cookie('yoo', 'biubiu')  # 设置cookie
        print(self.get_cookie('yoo'))  # 获取指定cookie

        name = self.get_argument('username')
        if name in ('year', 'mika'):
            self.set_cookie('username', name)
            self.set_secure_cookie('secureusername', name)
        else:
            self.write('请登录')
{% endraw %}
{% endcodeblock %}
同样在js中也可以操作cookie,jquery.cookie中可以更优雅的操作cookie.
{% codeblock lang:html cookie.html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <script>
        console.log(document.cookie); //获取所有cookie,是字符串, cookie的特性例如path 过期时间是不包含在内的
        document.cookie='yoo=666;path=/;'; //设置cookie,同时可以设置path和过期时间
        function setCookies(key,value,expires) {    //设置cookie并且按秒数设置过期时间
            var temp=[];
            var current_date=new Date();
            current_date.setSeconds(current_date.getSeconds()+5);
            document.cookie=key+"= "+value+";expires="+current_date.toUTCString(); //cookie的过期时间只接受UTCstring
        }

        //jquery的cookie插件 设置cookie
        $.cookie('yoo',666,{'path':'/','domain':'','expires':7})

    </script>
</body>
</html>
{% endraw %}
{% endcodeblock %}

#### CSRF
CSRF就是cross site request forgery,也就是跨站请求伪造,也就是仿造你网站中某些提交,我自己伪造提交到后台,已达到某种目的.
我们这里说CSRF是为了防止我们的网站遭到类似的攻击. Tornado自带了防止csrf的功能
首先要在设置里启用这个功能
{% codeblock lang:python setting.py %}
{% raw %}
settings = {
    "template_path": "Template",  # 模板路径的配置
    "static_path": "Static",  # 必须配置,否者静态文件永远找不到
    "static_url_prefix": "/biubiu/",  # 静态文件路径前缀
    "ui_methods": md,  # 配置ui method.
    "ui_modules": mt,  # 配置ui module.
    "cookie_secret": 'asdfasdfasdfasdfasdf',  # cookie加密
    "xsrf_cookies": True  # 跨站请求伪造防止
}
{% endraw %}
{% endcodeblock %}

{% codeblock lang:python handler.py %}
{% raw %}
class csrfHandler(baseHandler):
    def get(self, *args, **kwargs):
        self.render('csrf.html')

    def post(self, *args, **kwargs):
        self.write('post')
{% endraw %}
{% endcodeblock %}

{% codeblock lang:python csrf.html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>csrf</title>
</head>
<body>
    <form action="/csrf" method="post">
        {% raw xsrf_form_html() %}
        <p><input name="user" type="text" placeholder="用户名" /></p>
        <p><input name="pwd" type="password" placeholder="密码" /></p>
        <p>
            <input name="code" type="text" placeholder="验证码" />
            <img id="checkimg" onclick="changeCode();" style="cursor: pointer" src="/checkcode">
        </p>
        <input type="submit" value="Submit" />
    </form>
    <script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
    <input type="button" value="Ajax_CSRF" onclick="SubmitCsrf();"/>
    <script>
        function changeCode() {
            var code = document.getElementById('checkimg');
            code.src += '?'
        }

        function getCookie(name) {
            var r = document.cookie.match("\\b"+name+"=([^;]*\\b)");
            return r?r[1]:undefined;
        }

        function SubmitCsrf() {
            var id = getCookie("_xsrf");
            $.post({
                url:'/csrf',
                data:{'k1':'v1',"_xsrf":id},
                success:function (callback) {
                    //ajax请求发送成功后自动执行
                    //callback就是服务器write的数据 post
                    console.log(callback)

                }
            })
        }
    </script>
</body>
</html>
{% endraw %}
{% endcodeblock %}
这里有个梗....我们看到Tornado中用的是xsrf而不是csrf,其实他们指的是一个东西就是跨站请求伪造.当天因为有个同样的安全隐患的简称和csrf冲突了(好像是css的一个安全隐患,简称也是csrf)所以把第一个c改成了x....2333333
预防的原理也很简单.我们打开这个功能后,服务器会自动写入一个_xsrf的cookie.我们在提交表单的时候拿到这个值传回,然后服务器会自动比较他们.

#### 上传文件
1. 用form表单上传..这个最为简单.一个post请求到后台,利用tornado的方法去取到就可以了
2. AJAX上传,在某些特殊的情况下(当然是老版本ie啦)想异步上传就可以利用iframe来实现
{% codeblock lang:html fileupload.html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ul>
        {% for item in img_list %}
            <li><img src="/Static/{{item}}" /></li>
        {% end %}
    </ul>

    <input type="file" id="img" />
    <input type="button" onclick="fileupload();" >

    <form action="/fileupload" method="post" enctype="multipart/form-data">
        <input type="text" name="name" placeholder="你的名字">
        <h3>爱好</h3>
        <input name="favor" type="checkbox" value="1" />篮球:
        <input name="favor" type="checkbox" value="2" />足球:
        <input name="favor" type="checkbox" value="3" />乒乓球:
        <input type="file" name="file" />
        <input type="submit" value="提交">
    </form>

    <form id="my_form" name="form" action="/fileupload" method="post" enctype="multipart/form-data">
        <div id="main" >
            <input name="file" id="my_file" type="file" />
            <input type="button" name="action" value="upload" onclick="redirect();"/>
            <iframe id="my_iframe" name="'my_iframe" src="" style="display: none"></iframe>
        </div>
    </form>
    <script src='{{ static_url("jquery-3.2.0.min.js") }}'></script>
    <script>
        function fileupload() {
            //获取文件对象
            var fileObj = document.getElementById('img').files[0];
            var form = new FormData();
            form.append('name','v1');
            form.append('file',fileObj);
            var xhr = new XMLHttpRequest();
            xhr.open('post','/fileupload',true);
            xhr.send(form);
        }
        function jqfileUpload() {
            var fileObj = $("#img")[0].files[0];
            var form = new FormData();
            form.append('file',fileObj);
            $.ajax({
                type:'post',
                url:'/fileupload',
                data:form,
                processData:false, //告诉jq不要处理数据
                contentType:false, //告诉jq不要设置contentType
                success:function (arg) {
                    console.log(arg);
                }
            })
        }

        function redirect() {
            document.getElementById('my_iframe').onload=testt;
            document.getElementById('my_form').target='my_iframe';
            document.getElementById('my_form').submit();
        }
        function testt(ths) {
            var t = $('#my_iframe').content.find('body').text();
            console.log(t);
        }
    </script>
</body>
</html>
{% endraw %}
{% endcodeblock %}

{% codeblock lag:python Handler.py %}
{% raw %}
#文件上传相关
IMG_LIST=[]
class fileuploadHandler(baseHandler):
    def get(self, *args, **kwargs):
        self.render('fileupload.html',img_list=IMG_LIST)
    def post(self, *args, **kwargs):
        import os
        print(self.get_argument('name'),None)
        print(self.get_arguments('favor'),None)
        #print(self.get_argument('file'))
        file_metas = self.request.files['name']
        for meta in file_metas:
            file_name = meta['filename']
            with open(os.path.join('static',file_name),'wb') as up:
                up.write(meta['body'])
            IMG_LIST.append(file_name)
{% endraw %}
{% endcodeblock %}

#### 验证码
验证码的原理就是后台生成带有随机字符串的图片,然后将内容通过img标签输出到页面.同时将值保存在session中.
代码太多..github上看吧...

#### 异步非阻塞
坑太大...回头再填吧...

------

### 自定义Web组件

#### Session
首先Session这个东西,,很多Web框架都在用,并且自带实现,但是Tornado是一个轻量级框架,并没有实现Session这个东西.
Session是基于cookie实现的,原理是只需要在客户端的cookie上存储一个sessionID,这个唯一ID标识了服务器上的一块存储区域,里面有和该客户交互所需的各种各样的信息.
Session在服务器端的存储是门艺术,有的简单粗暴直接存内存,有的用分布式,有的启用缓存...
我们要实现Session组件的目标是,可以无缝的简单粗暴的使用session类似下面这样    **面向对象的思想去实现**
{% codeblock lang:python sessionHandelr.py %}
{% raw %}
# session 操作
class sessionHandler(baseHandler):
    def get(self):
        if self.get_argument('username', None) in ['year', 'mika']:
            self.session['is_login'] = True
        else:
            self.write('请登录')
{% endraw %}
{% endcodeblock %}
我们看到 只需要调用 `self.session[]`就可以轻松的放置和取出session中的数据
首先我们要有一个baseHandler类供我们以后的Handler类继承,在这个类里面,我们要做一些简单的初始化操作,比如初始化一个我们的Session类对象.
{% codeblock lang:python baseHaandler %}
class baseHandler(tornado.web.RequestHandler):
    def initialize(self):
        self.session = Session(self)
{% endcodeblock %}
接下来就是我们的Session类了,,想要像字典一样去调用我们需要用到`__setitem__`和`__getitem__`,剩下的无非就是生成随机sessionID和判断是否有SessionID的问题了
{% codeblock lang:python %}
{% raw %}
class Session:
    def __init__(self, handler):
        self.handler = handler
        self.random_str = None

    def __get_random_str(self):
        import hashlib
        import time
        obj = hashlib.md5()
        obj.update(bytes(str(time.time()), encoding='utf-8'))
        random_str = obj.hexdigest()
        return random_str

    def __setitem__(self, key, value):
        if not self.random_str:
            random_str = self.handler.get_cookie('__biubiu__', None)
            if not random_str:
                random_str = self.__get_random_str()
                SESSION[random_str] = {}
            else:
                if random_str in SESSION.keys():
                    pass
                else:
                    random_str = self.__get_random_str()
                    SESSION[random_str] = {}
            self.random_str = random_str
        SESSION[self.random_str][key] = value
        self.handler.set_cookie('__biubiu__', self.random_str)

    def __getitem__(self, key):
        random_str = self.handler.get_cookie('__biubiu__', None)
        if not random_str:
            return None
        usr_info_dict = SESSION.get(random_str, None)
        if not usr_info_dict:
            return None
        value = usr_info_dict.get(key, None)
        return value
{% endraw %}
{% endcodeblock %}

#### 表单验证
表单验证是所有Web程序都绕不开的问题了,因为前端的验证是可以跳过的,所以后端的验证是必不可少的
我们这里实现表单验证,同样要用**面向对象的思想**
我们的目的是我们后台验证表单的时候只需要调用我们穿件的一个前台Form的对应后台对象的验证方法就可以完成所有的验证并拿到结果
{% codeblock lang:python checkFormHandler %}
class checkFormHandler(baseHandler):
    def get(self, *args, **kwargs):
        self.render("checkForm.html",error_dict=None)
    def post(self, *args, **kwargs):
        files=self.request.files.get('file',[])

        obj=IndexForm()
        is_valid,success_dict,error_dict=obj.check_valid(self)
        if is_valid:
            print('success',success_dict)
            obj.file.save()
        else:
            print('error',error_dict)
            self.render('checkForm.html',error_dict=error_dict)
{% endcodeblock %}
我们在创建前台Form的对应后台类的时候只需要在后台类中将对应的字段类添加进来就可以,,例如我要输入一个ip,一个爱好,一个文件上传,在创建这些字段的时候我们可以自定义是否必填,和格式错误提示语等等自定义内容.
{% codeblock lang:python indexForm.py %}
class IndexForm(BaseForm):
    def __init__(self):
        self.ip=IPFiled(required=True,error_dict={'required':'不能填空啊'})
        self.favor = checkboxFiled(required=True, error_dict={'required': '不能填空啊'})
        self.file=fileFiled(required=True, error_dict={'required': '不能填空啊'})
{% endcodeblock %}
这个对应Form类有一个父类,里面有验证的方法,验证方法就循环调用内部变量的验证方法.
{% codeblock lang:python baseForm.py %}
class BaseForm:
    def check_valid(self,handle):
        flag=True
        value_dict={}
        error_message_dict={}
        success_value_dict={}
        for key,regular in self.__dict__.items():
            if type(regular)==checkboxFiled:
                input_value=handle.get_arguments(key)
            elif type(regular)==fileFiled :
                #获取文件名
                file_list=handle.request.files.get(key)
                input_value=[]
                for item in file_list:
                    input_value.append(item['filename'])
            else:
                input_value=handle.get_argument(key)
            regular.validate(key,input_value)
            if regular.is_valid:
                success_value_dict[key]=regular.value
            else:
                flag=False
                error_message_dict[key]=regular.error

            value_dict[key]=input_value
        return flag,success_value_dict,error_message_dict
{% endcodeblock %}
剩下来就是各个字段的类定义,里面有各个字段的验证方法.
{% codeblock lang:python Filed.py %}
class IPFiled:
    REGULAR=""
    def __init__(self,error_dict=None,required=True):
        self.error_dict={}
        if error_dict:
            self.error_dict.update(error_dict)
        self.required=required
        self.error=None
        self.is_valid=False
        self.value=None

    def validata(self,name,input_vale):
        if not self.required:
            self.is_valid=True
            self.value=input_vale
        else:
            if not input_vale.strip():
                if self.error_dict.get('required',None):
                    self.error=self.error_dict['required']
                else:
                    self.error="%s is required" % name
            else:
                ret=re.match(self.REGULAR,input_vale)
                if ret:
                    self.is_valid=True
                    self.value=ret.group()
                else:
                    if self.error_dict.get('valid',None):
                        self.error=self.error_dict['valid']
                    else:
                        self.error="%s valid is wrong" % name

class checkboxFiled:
    def __init__(self,error_dict=None,required=True):
        self.error_dict = {}
        if error_dict:
            self.error_dict.update(error_dict)
        self.required = required
        self.error = None
        self.is_valid = False
        self.value = None
    def validata(self,name,input_value):
        if not self.required:
            self.is_valid=True
            self.value=input_value
        else:
            if not input_value:
                if self.error_dict.get('required',None):
                    self.error=self.error_dict['required']
                else:
                    self.error="%s is required" % name
            else:
                self.is_valid = True
                self.value = input_value

class fileFiled:
    REGULAR = ""
    def __init__(self,error_dict=None,required=True):
        self.error_dict = {}
        if error_dict:
            self.error_dict.update(error_dict)
        self.required = required
        self.error = None
        self.is_valid = False
        self.value = None
    def validata(self,name,all_file_name_list):
        self.name=name
        if not self.required:
            self.is_valid=True
            self.value=all_file_name_list
        else:
            if not all_file_name_list:
                self.is_valid=False
                if self.error_dict.get('required',None):
                    self.error=self.error_dict['required']
                else:
                    self.error="%s is required" % name
            else:
                for file_name in all_file_name_list:
                    ret = re.match(self.REGULAR, file_name)
                    if not ret:
                        self.is_valid=False
                        if self.error_dict.get('valid', None):
                            self.error = self.error_dict['valid']
                        else:
                            self.error = "%s valid is wrong" % name
                        break
                    else:
                        self.value.append(file_name)

    def save(self,request):
        file_metas = request.files.get(self.name)
        for meta in file_metas:
            file_name = meta['filename']
            with open(os.path.join('static', file_name), 'wb') as up:
                up.write(meta['body'])
            IMG_LIST.append(file_name)
{% endcodeblock %}