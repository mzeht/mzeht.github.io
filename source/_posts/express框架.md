title: "express框架"
date: 2016-03-17 12:08:02
author: mzeht
avatar: /images/favicon.png
categories:
 - Web前端
 - Nodejs
tags:  
- Express
---

#基础知识
##Express介绍
　　npm提供了大量的第三方模块，其中不乏许多Web框架，比如我们本章节要讲述的一个轻量级的Web框架 ——— Express。

　　Express是一个简洁、灵活的node.js Web应用开发框架, 它提供一系列强大的功能，比如：模板解析、静态文件服务、中间件、路由控制等等,并且还可以使用插件或整合其他模块来帮助你创建各种 Web和移动设备应用,是目前最流行的基于Node.js的Web开发框架，并且支持Ejs、jade等多种模板，可以快速地搭建一个具有完整功能的网站。

<!-- more -->
　　好，下面我们就开始吧！

NPM安装

```
 npm install express
```
获取、引用

```
var express = require('express');
var app = express();
```
通过变量“app”我们就可以调用express的各种方法了，好戏刚刚开始，继续加油吧！

##创建应用
　　认识了Express框架，我们开始创建我们的第一个express应用。

　　在我们的默认项目主文件app.js添加如下内容：

```
var express = require('express');
var app = express();
app.get('/', function (request, response) {  
   response.send('Hello World!');
});
 
app.listen(80);
```

　　说明：在后面课程学习中，我们会统一使用80端口用于监听请求。

　　添加完毕之后，通过右侧栏的“测试地址”来查看浏览器内容，当看到“Hello World!”内容就表明一个简单的express应用已经创建成功了。
　　
##get请求
　前面我们实现了一个简单的express应用，下面我们就开始具体讲述它的具体实现，首先我们先来学习Express的常用方法。

　　get方法 —— 根据请求路径来处理客户端发出的GET请求。

　　格式：app.get(path,function(request, response));

　　path为请求的路径，第二个参数为处理请求的回调函数，有两个参数分别是request和response，代表请求信息和响应信息。

如下示例：

```
var express = require('express');
var app = express();
 
app.get('/', function(request, response) {
   response.send('Welcome to the homepage!');
});
app.get('/about', function(request, response) {
   response.send('Welcome to the about page!');
});
app.get("*", function(request, response) {
    response.send("404 error!");
});
app.listen(80); 
```
　　上面示例中，指定了about页面路径、根路径和所有路径的处理方法。并且在回调函数内部，使用HTTP回应的send方法，表示向浏览器发送一个字符串。

　　参照以上代码，试试自己设定一个get请求路径，然后浏览器访问该地址是否可以请求成功。
　　
##all函数的基本用法
　　和get函数不同app.all()函数可以匹配所有的HTTP动词，也就是说它可以过滤所有路径的请求，如果使用all函数定义中间件，那么就相当于所有请求都必须先通过此该中间件。

格式：app.all(path,function(request, response));

如下所示，我们使用all函数在请求之前设置响应头属性。

```
var express = require("express");
var app = express();
 
app.all("*", function(request, response, next) {
    response.writeHead(200, { "Content-Type": "text/html;charset=utf-8" });      //设置响应头属性值
    next();
});
 
app.get("/", function(request, response) {
    response.end("欢迎来到首页!");
});
 
app.get("/about", function(request, response) {
    response.end("欢迎来到about页面!");
});
 
app.get("*", function(request, response) {
    response.end("404 - 未找到!");
});
 
app.listen(80);
```
　　上面代码参数中的“*”表示对所有路径有效，这个方法在给特定前缀路径或者任意路径上处理时会特别有用，不管我们请求任何路径都会事先经过all函数。
　　
##use基本用法1
use是express调用中间件的方法，它返回一个函数。

格式：app.use([path], function(request, response, next){});

可选参数path默认为"/"。

1.使用中间件

app.use(express.static(path.join(__dirname, '/')));
如上呢，我们就使用use函数调用express中间件设定了静态文件目录的访问路径(这里假设为根路径)。

2.如何连续调用两个中间件呢，如下示例：

```
var express = require('express');
var app = express();
 
app.use(function(request, response, next){
    console.log("method："+request.method+" ==== "+"url："+request.url);
    next();
});
 
app.use(function(request, response){
    response.writeHead(200, { "Content-Type": "text/html;charset=utf-8" });
    response.end('示例：连续调用两个中间件');
});
 
app.listen(80);
```
回调函数的next参数，表示接受其他中间件的调用，函数体中的next()，表示将请求数据传递给下一个中间件。

上面代码先调用第一个中间件，在控制台输出一行信息，然后通过next()，调用第二个中间件，输出HTTP回应。由于第二个中间件没有调用next方法，所以req对象就不再向后传递了。

##use基本用法2
use方法不仅可以调用中间件，还可以根据请求的网址，返回不同的网页内容，如下示例：

```
var express = require("express");
var app = express();
 
app.use(function(request, response, next) {
   if(request.url == "/") {
      response.send("Welcome to the homepage!");
   }else {
      next();
   }
});
 
app.use(function(request, response, next) {
   if(request.url == "/about") {
     response.send("Welcome to the about page!");
   }else {
     next();
   }
});
 
app.use(function(request, response) {
  response.send("404 error!");
});
app.listen(80);
```
上面代码通过request.url属性，判断请求的网址，从而返回不同的内容。

##回调函数
　　Express回调函数有两个参数，分别是request(简称req)和response(简称res)，request代表客户端发来的HTTP请求，response代表发向客户端的HTTP回应，这两个参数都是对象。示例如下:

```
function(req, res) {
 
});
```

##获取主机名、路径名
　　今天我们就先来学习如何使用req对象来处理客户端发来的HTTP请求。

1.req.host返回请求头里取的主机名(不包含端口号)。

2.req.path返回请求的URL的路径名。

如下示例：

```
var express = require('express');
var app = express();
 
app.listen(8000);

app.get("*", function(req, res) {
    console.log(req.path);
    //console.log(req.host);
    res.send("req.host获取主机名，req.path获取请求路径名!");
});
```
试一试在浏览器中输入任意一个请求路径，通过req查看主机名或请求路径。


http://localhost:8000/about

```
/about
/favicon.ico
```

##query基本用法
　　query是一个可获取客户端get请求路径参数的对象属性，包含着被解析过的请求参数对象，默认为{}。


```
var express = require('express');
var app = express();
 
app.get("*", function(req, res) {
    console.log(req.query.参数名);
    res.send("测试query属性!");
});
 
app.listen(80);
```

通过req.query获取get请求路径的对象参数值。

格式：req.query.参数名；请求路径如下示例：

>例1： /search?n=Lenka

```
req.query.n  // "Lenka"
```
>例2： /shoes?order=desc&shoe[color]=blue&shoe[type]=converse


```
req.query.order  // "desc"
 
req.query.shoe.color  // "blue"
 
req.query.shoe.type  // "converse"
```

试一试get请求一个带参数路径，使用“req.query.参数名”方法获取请求参数值。

```
app.get('/about',function(req,res){
    console.log(req.query.name);
    res.send("name="+req.query.name);
});

```
http://localhost:8000/about?name=mzeht&name=jj

name=mzeht,jj

##param基本用法
和属性query一样，通过req.param我们也可以获取被解析过的请求参数对象的值。

格式：req.param("参数名")；请求路径如下示例：

例1： 获取请求根路径的参数值，如/?n=Lenka，方法如下：

```
var express = require('express');
var app = express();
 
app.get("/", function(req, res) {
    console.log(req.param("n")); //Lenka
    res.send("使用req.param属性获取请求根路径的参数对象值!");
});
 
app.listen(80);
```

例2：我们也可以获取具有相应路由规则的请求对象，假设路由规则为 /user/:name/，请求路径/user/mike,如下：

```
app.get("/user/:name/", function(req, res) {
    console.log(req.param("name")); //mike
    res.send("使用req.param属性获取具有路由规则的参数对象值!");
});

```


PS：所谓“路由”，就是指为不同的访问路径，指定不同的处理方法。
如果没有相应参数的数据 req.param 返回值为undefined;

看了上面的示例，试一试使用req.param属性解析一个请求路径对象，并获取请求参数值。
##params基本用法
和param相似，但params是一个可以解析包含着有复杂命名路由规则的请求对象的属性。

格式：req.params.参数名；

例1. 如上课时请求根路径的例子，我们就可以这样获取，如下：

```
var express = require('express');
var app = express();
 
app.get("/user/:name/", function(req, res) {
    console.log(req.params.name); //mike
    res.send("使用req.params属性获取具有路由规则的参数对象值!");
});
 
app.listen(80);
```

查看运行结果，和param属性功能是一样的，同样获取name参数值。

例2：当然我们也可以请求复杂的路由规则，如/user/:name/:id，假设请求地址为：/user/mike/123，如下：

```
app.get("/user/:name/:id", function(req, res) {
    console.log(req.params.id); //"123"
    res.send("使用req.params属性复杂路由规则的参数对象值!");
});
```
对于请求地址具有路由规则的路径来说，属性params比param属性是不是又强大了那么一点点呢！

##send基本用法
send()方法向浏览器发送一个响应信息，并可以智能处理不同类型的数据。格式如下：

res.send([body|status], [body]);
1.当参数为一个String时，Content-Type默认设置为"text/html"。

```
res.send('Hello World'); //Hello World
```
2.当参数为Array或Object时，Express会返回一个JSON。

```
res.send({ user: 'tobi' }); //{"user":"tobi"}
res.send([1,2,3]); //[1,2,3]
```

3.当参数为一个Number时，并且没有上面提到的任何一条在响应体里，Express会帮你设置一个响应体，比如：200会返回字符"OK"。

```
res.send(200); // OK
res.send(404); // Not Found
res.send(500); // Internal Server Error
```
send方法在输出响应时会自动进行一些设置，比如HEAD信息、HTTP缓存支持等等。

ps:

```
Fri, 18 Mar 2016 11:02:43 GMT express deprecated req.param(name): Use req.params, req.body, or req.query instead at app.js:37:21
Fri, 18 Mar 2016 11:02:43 GMT express deprecated req.param(name): Use req.params, req.body, or req.query instead at app.js:38:21
Fri, 18 Mar 2016 11:02:43 GMT express deprecated res.send(status): Use res.sendStatus(status) instead at app.js:42:9
```

#准备登陆
##模板引擎
　　从本节课程开始我们要使用express框架实现一个简单的用户登陆功能，让我们先准备一下相关资源。

　　在nodejs中使用express框架，它默认的是ejs和jade渲染模板，今天我们就以ejs模板为例，讲述模板渲染网页模板的基础功能。

1.ejs模板安装方法

npm install ejs
2.目录下安装好了之后，如何调用呢，如下所示：

//指定渲染模板文件的后缀名为ejs
app.set('view engine', 'ejs');
默认ejs模板只支持渲染以ejs为扩展名的文件，可能在使用的时候会觉得它的代码书写方式很不爽还是想用html的形式去书写，该怎么办呢，这时就得去修改模板引擎了，也就会用到express的engine函数。

engine注册模板引擎的函数，处理指定的后缀名文件。

// 修改模板文件的后缀名为html
app.set( 'view engine', 'html' );
// 运行ejs模块
app.engine( '.html', require( 'ejs' ).__express );
"__express"，ejs模块的一个公共属性，表示要渲染的文件扩展名。

##静态资源
由于环境的限制，这里我们就不使用静态资源了，但是实际开发中我们肯定会用到，具体使用规则已在下面列出，可参考。

如果要在网页中加载静态文件（css、js、img），就需要另外指定一个存放静态文件的目录，当浏览器发出非HTML文件请求时，服务器端就会到这个目录下去寻找相关文件。

1.项目目录下添加一个存放静态文件的目录为public。

2.在public目录下在添加三个存放js、css、img的目录，相应取名为javascripts、stylesheets、images。

3.然后就可以把相关文件放到相应的目录下了。

4.比如，浏览器发出如下的样式表请求：

 <link href="/stylesheets/bootstrap.min.css" rel="stylesheet" media="screen">
服务器端就到public/stylesheets/目录中寻找bootstrap.min.css文件。

有了静态目录文件，我们还得在启动文件里告诉它这个静态文件路径，需要指定一下，如下所示：

```
app.use(express.static(require('path').join(__dirname, 'public')));
```
PS：express.static —— 指定静态文件的查找目录。

使用use函数调用中间件指定express静态访问目录，'public'就是我们我们新建的用来存放静态文件的总目录。

##添加视图
好，下面我们就来添加网页模板了，项目中我们会新建一个目录用来单独存放模板文件，这里我们就统一放到根路径上了。

下面开始新建index.html、login.html、home.html三个页面。

index.html页面参考内容如下：

```
<div style="height:400px;width:550px;margin:50px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="margin-left: 35px;">
# 首页


        <form action="#"  role="form" style="margin-top: 90px;margin-left: 60px;">
# 欢迎进入首页！
            <div style="margin-top: 145px;">
                <input type="button" value="登 陆" />
            </div>
        </form>
    </div>
</div>

```


login.html页面参考内容如下：

```
...
    <title>用户登录</title>
    <meta charset="utf-8">
    <script src="http://code.jquery.com/jquery-2.1.1.min.js"></script>
...
 
<div style="height:300px;width:350px;margin:100px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="width:200px;margin:auto;margin-top:50px;">
# 用户登录
        <form action="#"  role="form" method="post" >
            <input id="username" type="text" name="username" style="margin: 20px 0px;" />
            <input id="password" type="password" name="password" />
            <div style="margin-top:30px;margin-left:125px;">
                <input type="button" value="登 陆" />
            </div>
        </form>
    </div>
</div>

```
home.html页面参考内容如下：

```
<div style="height:400px;width:550px;margin:50px auto;margin-left:auto;border:solid 1px;background: rgb(246, 246, 253);">
    <div style="margin-left: 45px;">
# 主页
        <form action="#"  role="form" style="margin-top: 90px;">
# 登陆成功！
            <div style="margin-top: 145px;">
                <input  type="button" value="退 出" />
            </div>
        </form>
    </div>
</div>

```
和静态文件一样，我们也要设置views存放的目录，如下：
// 设定views变量，意为视图存放的目录

```
app.set('views', __dirname);
```
有了网页模板和指定目录，下面就可以访问它们了。

##访问视图
我们要如何对网页模板进行访问呢，这就要用到res对象的render函数了。

1.render函数，对网页模板进行渲染。

2.格式：res.render(view, [locals], callback);

3.参数view就是模板的文件名callback用来处理返回的渲染后的字符串，options、callback可省略，在渲染模板时locals可为其模板传入变量值，在模板中就可以调用所传变量了，在后面我们会讲述具体使用方法，也可先自行使用看其效果。

4.比如渲染我们刚刚添加的index.html页面，我们就可以在app.js中写入如下内容：

```

var express = require('express');
var app = express();
var path = require('path');
 
app.set('views', __dirname);
 
app.set( 'view engine', 'html' );
app.engine( '.html', require( 'ejs' ).__express );
 
app.get('/', function(req, res) {
    res.render('index');
});
 
app.listen(8000);

```
运行之后在测试地址我们就可以看到所渲染的index页面了，试一试其他页面是否也可渲染成功？

##redirect基本用法
redirect方法允许网址的重定向，跳转到指定的url并且可以指定status，默认为302方式。

格式：res.redirect([status], url);

例1：使用一个完整的url跳转到一个完全不同的域名。

```
res.redirect("http://www.hubwiz.com");
```

例2：跳转指定页面，比如登陆页，如下：

```
res.redirect("login");
```
后面我们开始实现登陆功能，先试一下redirect重定向，跳转到我们网站如何？

#实现登陆
##访问视图
前面我们已经添加了视图模板并学习了访问视图的方法，那我们就先回顾一下。

1.参考以下代码，地址栏访问这几个请求路径查看是否可以成功。

```

app.get('/', function(req, res) {
    res.render('index');
});
 
app.get('/login',function(req,res){
    res.render('login');
});
 
app.get('/home',function(req,res){
    res.render('home');
});
```
当浏览器看到了相应的视图页面就说明我们的代码是没有问题的，继续加油吧！

##用户登陆
前面我们学习了express的get请求方法，今天我们就学习它的post请求方法。

1.post方法 —— 根据请求路径来处理客户端发出的Post请求。

2.格式：app.post(path,function(req, res));

3.和get方法一样，path为请求的路径，第二个参数为处理请求的回调函数，req和res分别代表请求信息和响应信息。

4.例如处理login的post请求，如下示例

```
app.post('/login',function(req,res){
 
});
```
了解了post方法，下面我们就开始使用post来实现简单的用户登陆功能。

##body基本用法
实现登陆之前我们先来了解一个属性 —— body。

body属性解析客户端的post请求参数，通过它可获取请求路径的参数值。

格式：req.body.参数名；

下面我们就来测试body属性的功能，做一些准备工作。

修改login.html，为登陆按钮增加登陆事件。

```
<input type="button" onclick="login();" value="登 陆">
function login(){
   var username = $('#username').val();
   var data = { "username": username };
   $.ajax({
           url:'/login',
           type:'POST',
           data:data
          });
}

```
要想使用body属性解析post请求参数值，我们需要先安装和引用express的两个中间件body-parser和multer，具体方法如下：。
2.1安装

```
npm install body-parser
npm install multer
```
2.2引用和调用

```
var bodyParser = require('body-parser');
var multer = require('multer');
   ......
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(multer());
```
中间件body-parser和multer用于处理和解析post请求的数据。

##body基本用法(2)
到这里我们就可以测试post请求的body属性的简单用法了。

1.修改好之后的完整的文件app.js如下所示：

```
var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var multer = require('multer');
app.set('views', __dirname);
app.set( 'view engine', 'html' );
app.engine( '.html', require( 'ejs' ).__express );
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(multer());
app.get('/',function(req,res){
    res.render('login');
});
app.post("/login", function(req, res) {
    console.log("用户名称为：" + req.body.username);
});
app.listen(8000);
```
通过测试地址访问浏览器，试试录入任意一个用户名登录，查看通过body属性是否可以成功获取？

##准备登陆
接下来我们就开始实现登陆功能，让我们要先做一些准备工作，为相关按钮添加点击事件、链接。

1.修改index.html，增加登陆链接。
```
[登 录](login)
```
2.强化login页面的login方法，实现一个简单的post请求：


```
 function login(){
    var username = $('#username').val();
    var password = $('#password').val();
    var data = { "username": username, "password":password};
    $.ajax({
             url:'login',
             type:'POST',
             data:data,
             success:function(data,status){
                  if(status == 'success'){
                      location.href='home';
                    }
             },
             error:function(data,status,e){
                  if(status == "error"){
                       location.href='login';
                     }
                   }
               });
       }
```
网页模板准备已经就绪，下面我们开始修改启动文件app.js的内容。


##准备登陆
下面我们就开始修改app启动文件的内容。

1.修改post方法，这里假设数据库中用户名的名字为admin、密码为admin。

```
app.post('/login',function(req,res){
    var user={
        username:'admin',
        password:'admin'
    }
if(req.body.username==user.username&&req.body.password==user.password)
    { 
       res.send(200);
    }else{
       res.send( 404 );
    }
});
2.一个完整的启动文件app.js如下所示：

var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var multer = require('multer');
 
app.set('views', __dirname);
app.set( 'view engine', 'html' );
app.engine( '.html', require( 'ejs' ).__express );
 
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(multer());
 
app.get('/', function(req, res) {
    res.render('index');
});
app.get('/home',function(req,res){
    res.render('home');
});
app.get('/login',function(req,res){
    res.render('login');
});
app.post('/login',function(req,res){
    var user={
        username:'admin',
        password:'admin'
    }
 if(req.body.username==user.username&&req.body.password==user.password)      {
      res.send(200);
   }else{
      res.send( 404 );
   }
});
 
app.listen(80);
```
到这里，一个简单的Post登录就完成了，使用浏览器运行本地端口试试效果吧！

##访问控制
简单登陆部分按照我们的求已经完成了，但网站好些并不安全，反复测试我们发现，home.html页面本来是登陆以后才访问的，现在我们不需登陆，直接在浏览器输入也可访问，这样肯定是不能被允许的，那么我们还得再次对登陆功能进行强化。

1.login.html页面增加EJS模板变量<%- message %>保存登陆提示信息。

```
...
    <%- message %>
# 用户登录
...
```
2.home.html页面，登陆成功后跳转并传入用户名：

```

# 恭喜_<%= user.username %>_，登陆成功！
```

PS：使用EJS模板变量值使用<%= variable_name %>输出方式，字符串输出时默认经过escape转义编码。 当我们想要输出一些动态生成的HTML标签时可使用<%- variable_nam %>输出方式，这种方式不会被escape转义编码。

3.home.html页面添加退出链接，如下：
```
[退 出](logout)
```


##访问控制
修改好了模板页，下面开始修改启动文件app.js的内容。

1.安装模块express-session并引用，安装、引用不在讲述。

2.使用新模块进行访问时间限制，如下：

```
var session = require('express-session');
...
app.use(session({
    secret:'secret',
    resave:true,
    saveUninitialized:false,
    cookie:{
        maxAge:1000*60*10  //过期时间设置(单位毫秒)
    }
}));
```
3.app.js文件新增中间件并设置模板变量值，如下：

```
app.use(function(req, res, next){
    res.locals.user = req.session.user;
    var err = req.session.error;
    res.locals.message = '';
    if (err) res.locals.message = '<div style="margin-bottom: 20px;color:red;">' + err + '</div>';
    next();
});
```
res.locals对象保存在一次请求范围内的响应体中的本地变量值。

PS：注意，中间件的放置顺序很重要，等同于执行顺序。而且，中间件必须放在HTTP动词方法之前，否则不会执行。

4.增加logout路径处理(用户登陆退出)和index路径请求处理，如下：

```
app.get('/logout', function(req, res){
    req.session.user = null;
    req.session.error = null;
    res.redirect('index');
});
app.get('/index', function(req, res) {
    res.render('index');
});
```
5.修改home路径请求处理，如下：

```
app.get('/home',function(req,res){
    if(req.session.user){
        res.render('home');
    }else{
        req.session.error = "请先登录"
        res.redirect('login');
    }
});
```
5.修改路径为login的Post请求

```
app.post('/login',function(req,res){
    var user={
        username:'admin',
        password:'admin'
    }
 if(req.body.username==user.username&&req.body.password==user.password){
        req.session.user = user;
        res.send(200);
    }else{
        req.session.error = "用户名或密码不正确";
        res.send( 404 );
    }
});
```
好，全部修改完毕，再次访问测试地址试试效果吧！

#路由登陆

##路由介绍
路由 ————为不同的访问路径，指定不同的处理方法。 在app.js中我们指定了app.get、app.post的不同路径的多个路由规则，在实际开发应用中，也会碰到具有多个路由记录的情况，针对这个问题，我们就要对这些路由记录做分开处理，以便于管理。

我们还是在登陆例子的基础上做如下修改。

1.添加三个js文件，名称分别为login、home、logout。

2.login.js文件，添加如下内容：


```
module.exports = function ( app ) {
    app.get('/login',function(req,res){
        res.render('login');
    });
 
    app.post('/login',function(req,res){
        var user={
            username:'admin',
            password:'admin'
        }
        if(req.body.username==user.username&&req.body.password==user.password){
            req.session.user = user;
            res.send(200);
        }else{
            req.session.error = "用户名或密码不正确"
            res.send( 404 );
        }
    });
}
```
3.home.js文件，添加如下内容：

```
module.exports = function ( app ) {
    app.get('/home',function(req,res){
        if(req.session.user){
            res.render('home');
        }else{
            req.session.error = "请先登录"
            res.redirect('login');
        }
    });
}
```

##使用路由2
4.logout.js文件，添加如下内容：

```
module.exports = function ( app ) {
    app.get('/logout', function(req, res){
        req.session.user = null;
        req.session.error = null;
        res.redirect('index');
    });
}

```
5.app.js文件中请求路径为logout、home、login的代码就可以删除了

6.新增的三个文件我们改如何使用呢，很简单，如下所示：

```
require('./login')(app);
require('./home')(app);
require('./logout')(app);
```
对，就是这么简单，我们只需引用路由的目录即可。

到这里我们的用户登陆功能就安全了，现学现用赶快试试效果吧！





