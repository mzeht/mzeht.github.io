title: "Nodejs入门"
date: 2016-03-15 19:38:32
author: mzeht
avatar: /images/favicon.png
categories:
 - Web前端
 - Nodejs
tags:
 -  Nodejs
---
总结<http://www.hubwiz.com/course/5359f6f6ec7452081a7873d8/>
#快速入门
##hello world
```
console.log('hello world');
```
输出

```
> hello world
```
##回调函数
由于node是一个异步事件驱动的平台，所以在代码中我们经常需要使用回调函数。下面是回调函数应用的经典示例：

```
setTimeout(function(){
    console.log('callback is called');
},2000);

```
我们传给setTimeout函数传入了一个匿名函数和一个调用时间2000(毫秒)，运行程序后等待2秒，可以看到输出了"callback is called"。
##标准回调函数

node.js中回调函数格式是约定俗成的，它有两个参数，第一个参数为err，第二个参数为data，顾名思义，err是错误信息，data则是返回的数据，示例如下:

```
function(err,data){
 
}
```
为了养成良好的编码习惯，在以后的课程示例或练习中，回调函数格式都将采用这样的格式。
##获取模块
为了支持快速开发，node平台上提供了大量的模块，封装了各自不同的功能，那么我们将如何调获取想要的模块呢，下来我们就来介绍。 在node中，我们可以使用require函数，具体语法如下：

```
require("模块");
```
怎么样，是不是很简单很方便，通过require函数我们就可以获取相应模块进而使用它的任意功能了。
##使用模块
os模块可提供操作系统的一些基本信息，它的一些常用方法如下：

```
var os = require("os");
 
var result = os.platform(); //查看操作系统平台
           //os.release(); 查看操作系统版本
           //os.type();  查看操作系统名称
           //os.arch();  查看操作系统CPU架构
 
console.log(result);
```
#进程管理
##简介
process是一个全局内置对象，可以在代码中的任何位置访问此对象，这个对象代表我们的node.js代码宿主的操作系统进程对象。

使用process对象可以截获进程的异常、退出等事件，也可以获取进程的当前目录、环境变量、内存占用等信息，还可以执行进程退出、工作目录切换等操作。

下面我们一起来学习一下process对象的一些常用方法。

##cwd函数的基本用法
当我们想要查看应用程序当前目录时，可以使用cwd函数，使用语法如下：

```
process.cwd();
```

##chdir函数的基本用法
如果需要改变应用程序目录，就要使用chdir函数了，它的用法如下：

process.chdir("目录");

##stdout的基本用法
stdout是标准输出流，它是干什么的呢？请下看下面的示例：

```
console.log = function(d){
    process.stdout.write(d+'\n');
}
```
没错，它的作用就是将内容打印到输出设备上，console.log就是封装了它。

##stderr的基本用法
stderr是标准错误流，和stdout的作用差不多，不同的是它是用来打印错误信息的，我们可以通过它来捕获错误信息，基本使用方法如下：

```
process.stderr.write(输入内容);
```

##stdin的基本用法
stdin是进程的输入流,我们可以通过注册事件的方式来获取输入的内容，如下：

```
process.stdin.on('readable', function() {
  var chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write('data: ' + chunk);
  }
});
```
示例中的chunk就是输入流中的内容。

##exit函数的基本用法
如果你需要在程序内杀死进程，退出程序，可以使用exit函数，示例如下：

```
process.exit(code);
```
参数code为退出后返回的代码，如果省略则默认返回0；

##注册事件
前面讲到如何在输入流中打印信息，当我们需要获取stdout内容的时候应该怎么做呢？请看如下的示例：

```
process.stdout.on('data',function(data){
   console.log(data);
});
```
为stdout注册data事件，我们就可以拿到它输出的内容了。
##设置编码
在我们的输入输出的内容中有中文的时候，可能会乱码的问题，这是因为编码不同造成的，所以在这种情况下需要为流设置编码，如下示例：

```
process.stdin.setEncoding(编码);
 
process.stdout.setEncoding(编码);
 
process.stderr.setEncoding(编码);
```

##课程小结
process的常用方法就讲到这里了，如果你想了解更多可以参考以下地址：

<http://nodejs.cn/api/process>
<http://nodeapi.ucdok.com/#/api/process.html>

#文件I/O fs模块的基本用法

开发中我们经常会有文件I/O的需求，node.js中提供一个名为fs的模块来支持I/O操作，fs模块的文件I/O是对标准POSIX函数的简单封装。
##writeFile函数的基本用法
文件I/O，写入是必修课之一。fs模块提供writeFile函数，可以异步的将数据写入一个文件, 如果文件已经存在则会被替换。用法如下：

>例：fs.writeFile(filename, data, callback)

```
var fs= require("fs");
 
fs.writeFile('test.txt', 'Hello Node', function (err) {
   if (err) throw err;
   console.log('Saved successfully'); //文件被保存
});
```
 
数据参数可以是string或者是Buffer,编码格式参数可选，默认为"utf8"，回调函数只有一个参数err。

##appendFile函数的基本用法
writeFile函数虽然可以写入文件，但是如果文件已经存在，我们只是想添加一部分内容，它就不能满足我们的需求了，很幸运，fs模块中还有appendFile函数，它可以将新的内容追加到已有的文件中，如果文件不存在，则会创建一个新的文件。使用方法如下：

>例：fs.appendFile(文件名,数据,编码,回调函数(err));

```
var fs= require("fs");
 
fs.appendFile('test.txt', 'data to append', function (err) {
   if (err) throw err;
 
    //数据被添加到文件的尾部
    console.log('The "data to append" was appended to file!'); 
});
```
编码格式默认为"utf8"

##exists函数的基本用法
如何检查一个文件是否存在呢？我想exists函数可以帮助你，用法如下：

>例：fs.exists(文件，回调函数(exists));

```
exists的回调函数只有一个参数，类型为布尔型，通过它来表示文件是否存在。

var fs= require("fs");
 
fs.exists('/etc/passwd', function (exists) {
  console.log(exists ? "存在" : "不存在!");
});
```

##rename函数的基本用法
修改文件名称是我们经常会遇见的事情，rename函数提供修改名称服务：

```
var fs= require("fs");
 
fs.rename(旧文件，新文件，回调函数(err){
   if (err) throw err;
   console.log('Successful modification,');
});
```

##rename函数的基本用法
移动文件也是我们经常会遇见的，可是fs没有专门移动文件的函数，但是我们可以通过rename函数来达到移动文件的目的，示例如下。

```
var fs = require('fs');
 
fs.rename(oldPath,newPath,function (err) {
   if (err) throw err;
   console.log('renamed complete');
});
```

##readFile函数的基本用法
读取文件是最常用到的功能之一，使用fs模块读取文件语法如下：

>例：fs.readFile(文件,编码,回调函数);

```
var fs = require('fs');
 
fs.readFile(文件名, function (err, data) {
  if (err) throw err;
  console.log(data);
});
```
回调函数里面的data,就是读取的文件内容。

##unlink函数的基本用法
面对一堆垃圾的文件总是有想删除的冲动，我有强迫症？你才有呢。

好在有unlink函数，终于得救了，示例如下：

>例：fs.unlink(文件,回调函数(err));

```
var fs = require('fs');
 
fs.unlink(文件, function(err) {
  if (err) throw err;
  console.log('successfully deleted');
});
```

##mkdir函数的基本用法
除了针对文件的操作，目录的创建、删除也经常遇到的，下面我们来看看node.js中如何创建目录：

>fs.mkdir(路径，权限，回调函数(err));
参数

路径：新创建的目录。
权限：可选参数，只在linux下有效，表示目录的权限，默认为0777，表示文件所有者、文件所有者所在的组的用户、所有用户，都有权限进行读、写、执行的操作。
回调函数：当发生错误时，错误信息会传递给回调函数的err参数。

##rmdir函数的基本用法
删除目录也是必不可少的功能，rmdir函数可以删除指定的目录：

>例：fs.rmdir(路径，回调函数(err));

var fs = require('fs');
 
fs.rmdir(path, function(err) {
  if (err) throw err;
  console.log('ok');
});

##readdir函数的基本用法
如果要读取目录下所有的文件应该怎么办呢？readdir函数可以读取到指定目录下所有的文件，示例如下：

```
var fs = require('fs');
 
fs.readdir(目录,回调函数(err,files));
```
回调函数 (callback) 接受两个参数 (err, files) 其中 files 是一个存储目录中所包含的文件名称的数组，数组中不包括 '.' 和 '..'。
##课程小结
文件I/O是最基本的操作，应该熟悉掌握。

fs模块不但提供异步的文件操作，还提供相应的同步操作方法，需要指出的是，nodejs采用异步I/O正是为了避免I/O时的等待时间，提高CPU的利用率，所以在选择使用异步或同步方法的时候需要权衡取舍。

本节课程讲解了fs模块常用的一些功能，当然它还有更多强大的功能，如果你想了解更多，可以参考以下资料：

<http://nodejs.cn/api/fs>

<http://nodeapi.ucdok.com/#/api/fs.html>

#url处理 url模块的基本用法
node.js为互联网而生，和url打交道是无法避免的了，url模块提供一些基础的url处理。

##parse函数的基础用法
parse函数的作用是解析url，返回一个json格式的数组，请看如下示例：

```
var url = require('url');
url.parse('http://www.baidu.com');
```
运行结果：

```
{ protocol: 'http:',
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: null,
  query: null,
  pathname: 'www.baidu.com',
  path: 'www.baidu.com',
  href: 'http://www.baidu.com' }
```

##parse函数 —— 条件解析
parse函数的第二个参数是布尔类型，当参数为true时，会将查询条件也解析成json格式的对象。

```
var url = require('url');
url.parse('http://www.baidu.com?page=1',true);
```
运行结果：

```
{ protocol: 'http:',
  slashes: true,
  auth: null,
  host: 'www.baidu.com',
  port: null,
  hostname: 'www.baidu.com',
  hash: null,
  search: '?page=1',
  query: { page: '1' },
  pathname: '/',
  path: '/?page=1',
  href: 'http://www.baidu.com/?page=1' }
```
试一试，当参数分别为true和false时，返回数据中的query字段的内容有什么不同。


##parse函数 —— 解析主机
parse函数的第三个参数也是布尔类型的，当参数为true，解析时会将url的"//"和第一个"/"之间的部分解析为主机名，示例如下：

```
var url = require('url');
url.parse('http://www.baidu.com/news',false,true);
```
运行结果：

```
{ protocol: 'http:',
  slashes: true,
  auth: null,
  host: 'www.baidu.com',
  port: null,
  hostname: 'www.baidu.com',
  hash: null,
  search: null,
  query: null,
  pathname: '/news',
  path: '/news',
  href: 'http://www.baidu.com/news' }
```
##format函数的基础用法
format函数的作用与parse相反，它的参数是一个JSON对象，返回一个组装好的url地址，请看如下示例：

```
var url = require('url');
url.format({
protocol: 'http:',
hostname:'www.baidu.com',
port:'80',
pathname :'/news',
query:{page:1}
});
```
运行结果：

```
http://www.baidu.com/news?page=1
```
参数JSON对象的字段跟parse函数解析后返回的JSON字段一一对应。

试一试传入一个对象，看看返回url是什么。


##resolve函数的基础用法
resolve函数的参数是两个路径，第一个路径是开始的路径或者说当前路径，第二个则是想要去往的路径，返回值是一个组装好的url，示例如下：

```
var url = require('url');
url.resolve('http://example.com/', '/one')  // 'http://example.com/one'
url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
```
动动手，试一试resolve函数。

##课程小结
本节课程讲解了url模块常用的一些功能，如果你想了解更多，可以参考以下资料：

<http://nodejs.cn/api/url>

#path优化 path模块的基本用法
本模块包含一套用于处理和转换文件路径的工具集,用于处理目录的对象，提高用户开发效率，让我们一起来快速的认识path对象的一些常用方法吧！

##normalize函数的基本用法
normalize函数将不符合规范的路径经过格式化转换为标准路径,解析路径中的.与..外，还能去掉多余的斜杠。

如下示例：

```
var path = require('path');  
var data = path.normalize('/path///normalize/hi/..');
console.log(data);
```
运行结果：

```
'/path/normalize/'
```
看了上面的示例，是不是感觉很So easy，赶快寻个路径试试效果吧(按步操作哟)！

##join函数的基本用法
join函数将传入的多个路径拼接为标准路径并将其格式化，返回规范后的路径，避免手工拼接路径字符串的繁琐. 如下示例：

```
var path = require('path');
var data = path.join('///you', '/are', '//beautiful');
console.log(data);
```
运行结果：

```
'/you/are/beautiful'
```

##dirname函数用来返回路径中的目录名. 如下示例：

```
var path = require('path');
var data = path.dirname('/foo/strong/cool/nice'); 
console.log(data);
```
运行结果：

```
'/foo/strong/cool'
```


##basename函数的基本用法
basename函数可返回路径中的最后一部分，并且可以对其进行条件排除. 如下示例：

例1：path.basename('路径字符串');

例2：path.basename('路径字符串', '[ext]')<排除[ext]后缀字符串>;

```
var path = require('path');    
var data1 = path.basename('/foo/strong/basename/index.html');
var data2 = path.basename('/foo/strong/basename/index.html','.html');
console.log(data1 + ' "and" ' + data2);
```
运行结果：

```
'index.html "and" index'
```

##extname函数的基本用法
extname函数返回路径中文件的扩展名(以最后一个'.'开始,返回'.'以及'.'以后的所有字符串,如没有'.',则返回空字符串). 如下示例：

```
var path = require('path');
var data = path.extname('index.html');
console.log(data);
```
运行结果：

```
'.html'
```

##课程小结
很高兴您学完了本节课程，关于path的一些常用方法就先讲到这里了，如果您还想了解更多的话,可以参考以下地址：

<http://nodejs.cn/api/path> 

<http://nodeapi.ucdok.com/#/api/path.html>

#字符串转换 Query String模块的基本介绍
Query String模块用于实现URL参数字符串与参数对象之间的互相转换，提供了"stringify"、"parse"等一些实用函数来针对字符串进行处理，通过序列化和反序列化，来更好的应对实际开发中的条件需求，对于逻辑的处理也提供了很好的帮助，下面就让我们一起来了解学习它吧！


##stringify函数的基本用法
stringify函数的作用就是序列化对象，也就是说将对象类型转换成一个字符串类型（默认的分割符（"&"）和分配符（"="）），本节先介绍它的基本用法，在下一节里我们将学习如何替换默认分配符，下面我们就通过以下例子来清楚的认识一下吧！

>例1：querystring.stringify("对象")

```
var querystring= require('querystring');
var result = querystring.stringify({foo:'bar',cool:['xux', 'yys']});
console.log(result);
```
运行结果：

```
foo=bar&cool=xux&cool=yys
```

##stringify函数的多参数用法
这节我们来学习stringify函数的多参数用法，上节我们知道了对象被序列化为字符串之后默认是通过分割符（"&"）和分配符（"="）组成的，那可不可以改变呢，这节我们就来了解一下，是否可以自己去定义组合结果，看下面的小例子

>例1：querystring.stringify("对象"，"分隔符"，"分配符")


```
var querystring = require('querystring');
var result = querystring.stringify({foo:'bar',cool:['xux', 'yys']},'*','$');
console.log(result);
```
运行结果：

```
'foo$bar*cool$xux*cool$yys'
```

##parse函数的基本用法
刚刚我们已经学习了stringify函数的作用，接下来就来学习反序列化函数——parse函数，parse函数的作用就是反序列化字符串（默认是由"="、"&"拼接而成），转换得到一个对象类型。如下示例：

>例1：querystring.parse("字符串")


```
var querystring = require('querystring');
var result = querystring.parse('foo=bar&cool=xux&cool=yys');
console.log(result);
```
运行结果：

```
{ foo: 'bar', cool: ['xux', 'yys']}
```

##parse函数的多参数用法
现在我们学习parse函数的扩展用法，和上节stringify函数的多参数用法不同的是，parse函数可以根据用户所自定义的分割符、分配符来反序列化字符串，从而得到相应的对象结果.如下示例：

>例1：querystring.parse("字符串"，"分隔符"，"分配符")


```
var querystring = require('querystring');
var result = querystring.parse('foo@bar$cool@xux$cool@yys','@','$');
console.log(result);
```
运行结果：

```
{ foo: '', bar: 'cool', xux: 'cool', yys: '' }
```

##课程小结
很高兴您学完了本节课程，关于querystring的一些常用方法就先讲到这里了，如果您还想了解更多的话,可以参考以下地址：

<http://nodejs.cn/api/querystring>
<http://nodeapi.ucdok.com/#/api/querystring.html>

#实用工具 UTIL模块的基本介绍
关于模块的如何访问前面已经讲述过了，相信大家已经完全掌握了，这节课我们来学习util模块。util模块呢，是一个Node.js核心模块，提供常用函数的集合，用于弥补核心JavaScript的一些功能过于精简的不足。并且还提供了一系列常用工具，用来对数据的输出和验证。

##inspect函数的基本用法
util.inspect(object,[showHidden],[depth],[colors])是一个将任意对象转换为字符串的函数，通常用于调试和错误输出。它至少接受一个参数object，即要转换的对象,我们来学习它的简单用法。使用语法如下：


```
var util = require('util');
var result = util.inspect(object);
console.log(result);
```
##format函数的基本用法
format函数根据第一个参数，返回一个格式化字符串，第一个参数是一个可包含零个或多个占位符的字符串。每一个占位符被替换为与其对应的转换后的值，支持的占位符有："%s(字符串)"、"%d(数字<整型和浮点型>)"、"%j(JSON)"、"%(单独一个百分号则不作为一个参数)"。

1：如果占位符没有相对应的参数，占位符将不会被替换.如示例：

```
var util = require('util');
var result = util.format('%s:%s', 'foo');
console.log(result);
```
运行结果：

```
'foo:%s'
```
2：如果有多个参数占位符，额外的参数将会调用util.inspect()转换为字符串。这些字符串被连接在一起，并且以空格分隔。如示例：

```
var util = require('util');
var result = util.format('%s:%s', 'foo', 'bar', 'baz');
console.log(result);
```
运行结果：

```
'foo:bar baz'
```
3：如果第一个参数是一个非格式化字符串，则会把所有的参数转成字符串并以空格隔开拼接在一块，而且返回该字符串。如示例：

```
var util = require('util');
var result = util.format(1, 2, 3);
console.log(result);
```
运行结果：

```
'1 2 3'
```

##isArray函数的基本用法
isArray函数可以判断对象是否为数组类型，是则返回ture,否则为false。语法如下：

```
var util = require('util');
var result = util.isArray(object);
console.log(result);
```

##isDate函数的基本用法
isDate函数可以判断对象是否为日期类型，是则返回ture,否则返回false。语法如下：

```
var util = require('util');
var result = util.isDate(object);
console.log(result);
```

##isRegExp函数的基本用法
isRegExp函数可以判断对象是否为正则类型，是则返回ture,否则返回false。语法如下：


```
var util = require('util');
var result = util.isRegExp(object);
console.log(result);
```

##课程小结
很高兴您学完了本节课程，关于util模块的一些常用方法就先讲到这里了，如果您还想了解更多的话,可以参考以下地址：

<http://nodejs.cn/api/util>

<http://nodeapi.ucdok.com/#/api/util.html>

#子进程 child_process模块的基本介绍
众所周知node.js是基于单线程模型架构，这样的设计可以带来高效的CPU利用率，但是无法却利用多个核心的CPU，为了解决这个问题，node.js提供了child_process模块，通过多进程来实现对多核CPU的利用. child_process模块提供了四个创建子进程的函数，分别是spawn，exec，execFile和fork。

##pawn函数的简单用法
spawn函数用给定的命令发布一个子进程，只能运行指定的程序，参数需要在列表中给出。如下示例：

```
var child_process = require('child_process');
var child = child_process.spawn( command );
child.stdout.on('data', function(data) {
  console.log(data);
});
```
通过执行命令得到返回结果，我们就可以拿到标准输出流数据了。

##exec函数的简单用法
exec也是一个创建子进程的函数，与spawn函数不同它可以直接接受一个回调函数作为参数，回调函数有三个参数，分别是err, stdout , stderr，基本使用方法如下：

var child_process = require('child_process');
child_process.exec( command , function(err, stdout , stderr ) {
  console.log( stdout );
});
exec函数可以直接接受一个回调函数作为参数，回调函数有三个参数，分别是err, stdout,stderr，非常方便直接使用，

##execFile函数的简单用法
execFile函数与exec函数类似，但execFile函数更显得精简，因为它可以直接执行所指定的文件，基本使用方法如下：

```
var child_process = require('child_process');
child_process.execFile( file , function(err, stdout , stderr ) {
  console.log( stdout );
});
```
execFile与spawn的参数相似，也需要分别指定执行的命令和参数，但可以接受一个回调函数，与exec的回调函数相同。


##execFile函数的简单用法
execFile函数与exec函数类似，但execFile函数更显得精简，因为它可以直接执行所指定的文件，基本使用方法如下：

```
var child_process = require('child_process');
child_process.execFile( file , function(err, stdout , stderr ) {
  console.log( stdout );
});
```
execFile与spawn的参数相似，也需要分别指定执行的命令和参数，但可以接受一个回调函数，与exec的回调函数相同。

##课程小结
恭喜你学完了本节课程，关于child_process的四个函数的基本简单用法，就先讲到这里了，如果您还想深入了解学习的话,可以参考以下地址：

<http://nodejs.cn/api/child_process>
<http://nodeapi.ucdok.com/#/api/child_process.html>


