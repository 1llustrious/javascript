千呼万唤始出来，来到了最重要的部分，nodejs是使用js语言进行web开发的，在代码审计的时候，难免会遇到nodejs。<br />nodejs就是js语言走向了服务端，用于web开发。
<a name="oS8hs"></a>
# 基本模块
<a name="EntSD"></a>
## global
```javascript
global.console
Console {
  log: [Function: bound ],
  info: [Function: bound ],
  warn: [Function: bound ],
  error: [Function: bound ],
  dir: [Function: bound ],
  time: [Function: bound ],
  timeEnd: [Function: bound ],
  trace: [Function: bound trace],
  assert: [Function: bound ],
  Console: [Function: Console] }
```
<a name="FEV1d"></a>
## process
表示进程的模块，经常用来进行rce操作。
```javascript
require('process')
console.log(process.version);
console.log(process.platform);
console.log(process.arch);
console.log(process.cwd());
/*v18.15.0
win32
x64
D:\webpath
v18.15.0
*/
```
<a name="sPgMD"></a>
# fs
Node.js内置的fs模块就是文件系统模块，负责读写文件，fs模块同时提供了异步和同步的方法。
```javascript
'use strict';
var fs = require('fs');
fs.readFile('sample.txt', 'utf-8', function (err, data) {
    if (err) {
        console.log(err);
    } else {
        console.log(data);
    }
});
```
回调函数接收err和data，分别表示出错和正常的回显内容。<br />二进制文件：
```javascript
'use strict';

var fs = require('fs');

fs.readFile('sample.png', function (err, data) {
    if (err) {
        console.log(err);
    } else {
        console.log(data);
        console.log(data.length + ' bytes');
    }
});
```
读文件时返回一个buffer对象，buffer对象就是由零个或多个bytes组成的数组。
<a name="OJZzk"></a>
## 同步读取
提供sync后缀，不接受回调函数，直接返回结果。
```javascript
'use strict';

var fs = require('fs');

var data = fs.readFileSync('sample.txt', 'utf-8');
console.log(data);
```
<a name="keOX0"></a>
## 写文件
fs.writeFile()，同样也有个同步写文件，writeFileSync'<br />stat()方法，用来返回文件的具体信息。
```javascript
'use strict';

var fs = require('fs');

fs.stat('sample.txt', function (err, stat) {
    if (err) {
        console.log(err);
    } else {
        // 是否是文件:
        console.log('isFile: ' + stat.isFile());
        // 是否是目录:
        console.log('isDirectory: ' + stat.isDirectory());
        if (stat.isFile()) {
            // 文件大小:
            console.log('size: ' + stat.size);
            // 创建时间, Date对象:
            console.log('birth time: ' + stat.birthtime);
            // 修改时间, Date对象:
            console.log('modified time: ' + stat.mtime);
        }
    }
});
```
<a name="XRMQg"></a>
# express
一个用于web后端的框架，使用起来如下
```javascript
var express = require('express');
var app = express();
app.get('/', function (req, res) {
    res.send('Hello World!');//send方法向用户端发送响应。
});

app.listen(3000, function () {
    console.log('Example app listening on port 3000!');
});
```
express()用于返回一个express对象，执行后续get操作，get接收路径和回调函数，用于处理<br />同样如果需要异步调用的话，就要写大量回调。
```javascript
app.get('/test', function (req, res) {
    fs.readFile('/file1', function (err, data) {
        if (err) {
            res.status(500).send('read file1 error');
        }
        fs.readFile('/file2', function (err, data) {
            if (err) {
                res.status(500).send('read file2 error');
            }
            res.type('text/plain');
            res.send(data);
        });
    });
});
```
<a name="kaYWd"></a>
# koa
```javascript
const Koa = require('koa');


const app = new Koa();

// 对于任何请求，app将调用该异步函数处理请求：
app.use(async (ctx, next) => {
    await next();
    ctx.response.type = 'text/html';
    ctx.response.body = '<h1>Hello, koa2!</h1>';
});

// 在端口3000监听:
app.listen(3000);
console.log('app started at port 3000...');
```
导入的koa是class，然后对象化给app，通过use，接收一个回调函数并且执行。<br />在Koa框架中，next()是一个函数，用于将控制权转移到下一个中间件函数。当一个中间件函数调用next()时，Koa将暂停当前中间件的执行，并将控制权传递给下一个中间件，直到所有中间件函数都执行完毕，然后控制权返回给上一个中间件，直到最初的请求处理函数返回响应。<br />中间件就是一个函数，其可以访问请求对象（request）、响应对象（response）以及应用程序的请求-响应循环中的下一个中间件函数。中间件函数可以：<br />执行任何代码<br />修改请求对象（request）和响应对象（response）。<br />终结请求-响应循环。<br />调用下一个中间件函数。<br />很多async函数组成一个处理链，每个async函数都可以做一些自己的事情，然后用await next()来调用下一个async函数。我们把每个async函数称为middleware，这些middleware可以组合起来，完成很多有用的功能。<br />当执行到next时，轮到下个中间件执行。之后的操作要等其它中间件完成自己的操作再执行。
<a name="rAoWq"></a>
## koa-router
```javascript
const Koa = require('koa');
const Router = require('koa-router');

const app = new Koa();
const router = new Router();

router.get('/', async (ctx, next) => {
  ctx.body = 'Hello World!';
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);

```
对router进行的是函数调用，<br />定义了一个路由/用于返回helloworld。router.routes()用于user挂载路由中间件。<br />处理post请求时，可以使用router.post('/path', async fn)。<br />但由于koa本身不解析post表单，所以需要使用另外的中间件解析。
```javascript
const bodyParser = require('koa-bodyparser');
...
//加上
app.use(bodyParser());
```
<a name="cSxPa"></a>
# Nunjucks
nunjucks是一种模板引擎。<br />模板引擎就是基于模板配合数据构造出字符串输出的一个组件。<br />以下为一简单模板引擎：
```javascript
function examResult (data) {
    return `${data.name}同学一年级期末考试语文${data.chinese}分，数学${data.math}分，位于年级第${data.ranking}名。`
}
```
模板引擎把模板字符串里面对应的变量替换以后，进行输出。<br />简单逻辑
```javascript
{% if score >= 90 %}
    成绩优秀，应该奖励
{% elif score >=60 %}
    成绩良好，继续努力
{% else %}
    不及格，建议回家打屁股
{% endif %}
```
这里就会自动执行一个if逻辑判断，但是这种简单逻辑有时会造成ssti漏洞。<br />例子：
```javascript
// 引入Nunjucks模块
const nunjucks = require('nunjucks');
const express = require('express');
const app = express();

// 配置Nunjucks模板引擎
nunjucks.configure('views', {
  autoescape: true,
  express: app
});

// 创建路由，渲染HTML页面
app.get('/', (req, res) => {
  // 传递数据到模板中
  const data = {
    title: 'Welcome to my website!',
    content: 'This is my first web page using Nunjucks template engine.'
  };
  res.render('index.html', data);
});

// 监听端口
app.listen(3000, () => {
  console.log('Server started on port 3000...');
});

```
render用于将data渲染成html数据。<br />也可以创建nunjucks环境
```javascript
env = new nunjucks.Environment(
            new nunjucks.FileSystemLoader('views', {
                noCache: noCache,
                watch: watch,
            }), {
                autoescape: autoescape,
                throwOnUndefined: throwOnUndefined
            });
createEnv('views', {
    watch: true,
    filters: {
        hex: function (n) {
            return '0x' + n.toString(16);
        }
    }
});
```
之后就可以使用环境对象对其渲染。
<a name="Mj1a2"></a>
# MVC
Model-View-Controller<br />工程结构
```
view-koa/
|
+- .vscode/
|  |
|  +- launch.json <-- VSCode 配置文件
|
+- controllers/ <-- Controller
|
+- views/ <-- html模板文件
|
+- static/ <-- 静态资源文件
|
+- controller.js <-- 扫描注册Controller
|
+- app.js <-- 使用koa的js
|
+- package.json <-- 项目描述文件
|
+- node_modules/ <-- npm安装的所有依赖包
```
![](https://cdn.nlark.com/yuque/0/2023/png/34502958/1683125223552-c161bed6-d7bd-4804-bd4d-95afc4ed544c.png#averageHue=%23f9f9f9&clientId=u9a9d9089-8a75-4&from=paste&id=uc8b19b73&originHeight=382&originWidth=439&originalType=url&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&taskId=u3b804716-7f51-4e94-92be-47265bf8aad&title=)<br />app,js处理请求后，传给模板进行渲染，渲染后给用户端。<br />写好get函数处理
```javascript
async (ctx, next) => {
    ctx.render('index.html', {
        title: 'Welcome'
    });
}
```
post函数处理
```javascript
async (ctx, next) => {
    var
        email = ctx.request.body.email || '',
        password = ctx.request.body.password || '';
    if (email === 'admin@example.com' && password === '123456') {
        // 登录成功:
        ctx.render('signin-ok.html', {
            title: 'Sign In OK',
            name: 'Mr Node'
        });
    } else {
        // 登录失败:
        ctx.render('signin-failed.html', {
            title: 'Sign In Failed'
        });
    }
}
```
这里出现了三个html渲染。<br />一般来说，为了设计页面，会在外设计一个css文件，该文件由于是静态的，一般放在static下面。
```javascript
view-koa/
|
+- static/
   |
   +- css/ <- 存放bootstrap.css等
   |
   +- fonts/ <- 存放字体文件
   |
   +- js/ <- 存放bootstrap.js等   
```
在编写html时，直接src引用就行。<br />然后又要编写一个中间件，用来处理静态。
```javascript
const path = require('path');
const mime = require('mime');
const fs = require('mz/fs');

// url: 类似 '/static/'
// dir: 类似 __dirname + '/static'
function staticFiles(url, dir) {
    return async (ctx, next) => {
        let rpath = ctx.request.path;
        // 判断是否以指定的url开头:
        if (rpath.startsWith(url)) {
            // 获取文件完整路径:
            let fp = path.join(dir, rpath.substring(url.length));
            // 判断文件是否存在:
            if (await fs.exists(fp)) {
                // 查找文件的mime:
                ctx.response.type = mime.lookup(rpath);
                // 读取文件内容并赋值给response.body:
                ctx.response.body = await fs.readFile(fp);
            } else {
                // 文件不存在:
                ctx.response.status = 404;
            }
        } else {
            // 不是指定前缀的URL，继续处理下一个middleware:
            await next();
        }
    };
}

module.exports = staticFiles;
```
这里对url前缀判断，如果是指定前缀，则执行响应操作。不是则交给下一个中间件。<br />然后
```javascript
let staticFiles = require('./static-files');
app.use(staticFiles('/static/', __dirname + '/static'));
```
先是导入要使用的中间件文件，再给中间件函数传入对应的static参数
