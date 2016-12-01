
# [Nightmare](https://github.com/segmentio/nightmare)

> Nightmare的机翻文档。
    
Nightmare是一个高级别的自动化浏览器程序。  
  
目的是仅仅暴露一些简单的方法，并且有相对于脚本来说同步的API，而不是深层嵌套的回调。它专为在没有API的网站上自动执行任务而设计。  
  
在封面下，它使用[Electron](http://electron.atom.io/)，这是类似于[PhantomJS](http://phantomjs.org/)，但更快，更现代。  
  
[Daydream](https://github.com/segmentio/daydream)是由 [@stevenmiller888](https://github.com/stevenmiller888) 构建的一个互补的chrome扩展，在您浏览时为您生成Nightmare脚本。  
  
非常感谢[@matthewmueller](https://github.com/matthewmueller)和[@rosshinkley](https://github.com/rosshinkley)他们对Nightmare的帮助。
  
* [Examples](#examples)
* [API](#api)
    * [Set up an instance（创建实例）](#nightmareoptions)
    * [Interact with the page（与页面交互）](#interact-with-the-page)
    * [Extract from the page（从页面获取）](#extract-from-the-page)
    * [Cookies](#cookies)
    * [Proxies（代理）](#proxies)
    * [Extending Nightmare（扩展Nightmare）](#extending-nightmare)
* [Usage（用法）](#usage)
* [Debugging](#debugging)
* [Supplementary Examples and Documentation（补充实例和文档）](https://github.com/rosshinkley/nightmare-examples)  
 


 
  
## Examples

在Yahoo运行脚本：
```javascript
var Nightmare = require('nightmare');
var nightmare = Nightmare({ show: true });

nightmare
  .goto('http://yahoo.com')
  .type('form[action*="/search"] [name=p]', 'github nightmare')
  .click('form[action*="/search"] [type=submit]')
  .wait('#main')
  .evaluate(function () {
    return document.querySelector('#main .searchCenterMiddle li a').href
  })
  .end()
  .then(function (result) {
    console.log(result)
  })
  .catch(function (error) {
    console.error('Search failed:', error);
  });
```

你可以运行这个脚本:
```
npm install nightmare
node example.js
```

或者让我们进行Mocha测试：
```javascript
var Nightmare = require('nightmare');
var expect = require('chai').expect; // jshint ignore:line

describe('test yahoo search results', function() {
  it('should find the nightmare github link first', function(done) {
    var nightmare = Nightmare()
    nightmare
      .goto('http://yahoo.com')
      .type('form[action*="/search"] [name=p]', 'github nightmare')
      .click('form[action*="/search"] [type=submit]')
      .wait('#main')
      .evaluate(function () {
        return document.querySelector('#main .searchCenterMiddle li a').href
      })
      .end()
      .then(function(link) {
        expect(link).to.equal('https://github.com/segmentio/nightmare');
        done();
      })
  });
});
```

你可以在[这里](https://github.com/segmentio/nightmare/blob/master/test/index.js)看到每个函数的例子。  
  
请注意，这些示例使用Mocha的[mocha-generators](https://www.npmjs.com/package/mocha-generators)包，支持generators。  
  
### 安装依赖
```
npm install
```
   
### To run the mocha tests
```
npm test
```
  
### Node版本
Nightmare必须运行在NodeJS 4.x或更高版本上。




## API

### Nightmare(options)
创建一个可在网络上浏览器的新的实例。这里记录了[可用的选项](https://github.com/electron/electron/blob/master/docs/api/browser-window.md#new-browserwindowoptions)，以及以下Nightmare的特定选项。

### Nightmare.version
返回Nightmare的版本号。

#### waitTimeout (default: 30s)
如果`.wait()`在设置的时间范围内没有返回`true`，这将抛出异常。
```javascript
var nightmare = Nightmare({
  waitTimeout: 1000 // in ms
});
```

#### gotoTimeout (default: 30s)
如果由操作（例如，`.click()`）导致的页面转换未在设置的时间范围内完成，这将使Nightmare继续运行。如果`loadTimeout`小于`gotoTimeout`，则`gotoTimeout`抛出的异常将被禁止。
```javascript
var nightmare = Nightmare({
  loadTimeout: 1000 // in ms
});
```

#### executionTimeout (default: 30s)
等待`.evaluate()`语句完成的最长时间。
```javascript
var nightmare = Nightmare({
  executionTimeout: 1000 // in ms
});
```

#### paths
Electron知道的默认系统路径。 以下是可用路径的列表：
[https://github.com/electron/electron/blob/master/docs/api/app.md#appgetpathname](https://github.com/electron/electron/blob/master/docs/api/app.md#appgetpathname)  

你可以通过执行以下操作在Nightmare中覆盖它们：
```javascript
var nightmare = Nightmare({
  paths: {
    userData: '/user/data'
  }
});
```

#### switches
由Electron支持的Chrome浏览器使用的命令行开关。以下是受支持的Chrome命令行开关的列表：[https://github.com/electron/electron/blob/master/docs/api/chrome-command-line-switches.md](https://github.com/electron/electron/blob/master/docs/api/chrome-command-line-switches.md)
```javascript
var nightmare = Nightmare({
  switches: {
    'proxy-server': '1.2.3.4:5678',
    'ignore-certificate-errors': true
  }
});
```

#### electronPath
预创建二进制的Electron路径。这对于在不同版本的Electron上进行测试很有用。注意，Nightmare只支持这个包的版本。使用此选项请自行承担风险。
```javascript
var nightmare = Nightmare({
  electronPath: require('electron')
});
```

#### dock (OS X)
（可选）在dock中显示Electron图标的布尔值（默认为`false`）。这对于测试目的很有用。
```javascript
var nightmare = Nightmare({
  dock: true
});
```

#### openDevTools
（可选）在窗口中显示DevTools（开发者工具），使用`true`，或者在单独的窗口中显示使用包含`mode: 'detach'`的哈希对象。哈希将传递到`contents.openDevTools()`以进行处理。这也用于测试目的。请注意，只有将`show`设置为`true`，才会使用此选项。
```javascript
var nightmare = Nightmare({
  openDevTools: {
    mode: 'detach'
  },
  show: true
});
```

#### typeInterval (default: 100ms)
使用`.type()`时，在按键之间等待多长时间。
```javascript
var nightmare = Nightmare({
  typeInterval: 20
});
```

#### pollInterval (default: 250ms)
在检查`.wait()`条件是否成功之前等待多长时间。
```javascript
var nightmare = Nightmare({
  pollInterval: 50 //in ms
});
```

#### maxAuthRetries (default: 3)
定义在使用`.authenticate()`设置时重试身份验证的次数。

#### .engineVersions()
获取Electron和Chromium的版本。

#### .useragent(useragent)
设置由Electron使用的用户代理。

#### .authentication(user, password)
设置使用基本身份验证访问网页的用户`user`和密码`password`。 请务必在调用`.goto(url)`之前设置它。

#### .end()
完成任何队列操作，断开和关闭Electron进程。请注意，如果您使用promises，则`.then()`必须在`.end()`之后调用以运行`.end()`任务。

#### .halt(error, done)
清除所有排队的操作，杀死Electron进程，并通过错误消息或"Nightmare halted"到一个未解决的Promise。 完成将在过程退出后调用。


### Interact with the Page

#### .goto(url[, headers])
加载一个网址。（可选），可以设置`goto`的`header`（请求头）。当页面加载成功时，goto返回包含有关页面加载的元数据的对象，包括：
* `url`：加载的URL
* `code`：HTTP状态代码（例如200，404，500）
* `method`: 使用的HTTP方法（例如"GET"，"POST"）
* `referrer`：窗口在此载入之前显示的页面或空白字符串（如果这是首次加载页面）。
* `headers`：表示请求的响应头的对象，如`{header1-name：header1-value，header2-name：header2-value}`
如果页面加载失败，则错误将是具有以下属性的对象：
* `message`：描述错误类型的字符串
* `code`：底层的错误代码描述错误原因。注意这不是HTTP状态代码。对于可能的值，参考[https://code.google.com/p/chromium/codesearch#chromium/src/net/base/net_error_list.h](https://code.google.com/p/chromium/codesearch#chromium/src/net/base/net_error_list.h)
* `details`：包含错误的其他详细信息的字符串。这可以是null或空字符串。
* `url`：无法加载的网址。

请注意，来自服务器的任何有效响应都被视为“成功”。这意味着类似404 "not found" 错误是`goto`的成功结果。只有不会导致页面出现在浏览器窗口中的事情才是错误，例如在给定地址没有服务器响应，服务器挂在响应中间，或无效的网址。  
您还可以通过在Nightmare构造函数上设置[`gotoTimeout` option](#gotoTimeout (default: 30s)来调整`goto`在超时前等待的时间。

#### .back()
返回上一页。

#### .forward()
前进到下一页。

#### .refresh()
刷新当前页面。

#### .click(selector)
单击`selector` element（选择器）一次。

#### .mousedown(selector)
鼠标按下`selector` element（选择器）一次。

#### .mouseover(selector)
鼠标进入`selector` element（选择器）一次。

#### .type(selector[, text])
输入提供给`selector`的`text`。为`text`提供的空值或false值将清除选择器的值。`.type()`模仿用户在文本框中键入，并执行正确的键盘事件。   
按键也可以使用带有`.type()`的Unicode值触发。例如，如果你想按下一个回车键，你会写`.type('document'，'\ u000d')`。
> 如果你不需要键盘事件，考虑使用`.insert()`，因为它会更快，更稳定。

#### .insert(selector[, text])
类似于`.type()`，`.insert()`提供输入到`selector`中的文本。为`text`提供的空值或false值将清除选择器的值。`.insert()`比`.type()`快，但不会触发键盘事件。

#### .check(selector)
选中`selector`复选框。

#### .uncheck(selector)
取消选中`selector`复选框。

#### .select(selector, option)
将`selector`下拉列表元素更改为带有属性[value = `option`]的选项

#### .scrollTo(top, left)
将页面滚动到所需位置。`top`和`left`始终相对于文档的左上角。

#### .viewport(width, height)
设置视口大小。


#### .inject(type, file)
将本地文件注入到当前页面，文件类型必须是`js`或`css`。

#### .evaluate(fn[, arg1, arg2,...])
使用`arg1，arg2，...`调用页面上的`fn`。完成后，它返回`fn`的返回值。用于从页面提取信息。例子：
```javascript
var selector = 'h1';
nightmare
  .evaluate(function (selector) {
    // now we're executing inside the browser scope.
    return document.querySelector(selector).innerText;
   }, selector) // <-- that's how you pass parameters from Node scope to browser scope
  .then(function(text) {
    // ...
  })
```

回调支持作为evaluate的一部分。如果传递的参数比计算函数预期的参数少一个，evaluation将传递一个回调作为该函数的最后一个参数。例如：
```javascript
var selector = 'h1';
nightmare
  .evaluate(function (selector, done) {
    // now we're executing inside the browser scope.
    setTimeout(() => done(null, document.querySelector(selector).innerText), 2000);
   }, selector)
  .then(function(text) {
    // ...
  })
```

注意，回调只支持一个值参数。最终，回调将被包装在原生的Promise中，并且只能解析单个值。   
Promises支持作为evaluate的一部分。如果函数的返回值有一个`then`成员，假设`.evaluate()`在等待Promise。例如：
```javascript
var selector = 'h1';
nightmare
  .evaluate(function (selector, done) {
      return new Promise((resolve, reject) => {
        setTimeout(() => resolve(document.querySelector(selector).innerText), 2000);
   }, selector)
  .then(function(text) {
    // ...
  })
```

#### .wait(ms)
等待多少毫秒，例如`.wait(5000)`

#### .wait(selector)
等待直到`selector`存在，例如`.wait('#pay-button')`。

#### .wait(fn[, arg1, arg2,...])
等待直到`fn`在页面上运行并带上参数`arg1, arg2,...`完毕并且返回`true`。所有的参数都是可选的。
有关使用情况，请参见`.evaluate()`。

#### .header([header, value])
添加一个`header`来覆盖HTTP请求头，如果`headers`未定义，则`headers`的覆盖将被重置。
> Add a header override for all HTTP requests. If header is undefined, the `header` overrides will be reset.


### Extract from the Page

#### .exists(selector)
返回页面上是否存在`selector`。

#### .visible(selector)
返回`selector`是否可见。

#### .on(event, callback)
使用回调捕获页面事件。您必须在调用`.goto()`之前调用`.on()`。支持的事件在[这里](http://electron.atom.io/docs/api/browser-window/)记录。

##### Additional "page" events
###### .on('page', function(type="error", message, stack))
如果在网页上抛出任何JavaScript异常，则会触发此事件。但是，如果注入的JavaScript代码（例如通过`.evaluate()`）抛出异常，则不会触发此事件。

##### "page" events
监听例如`window.addEventListener('error'), alert(...), prompt(...)` & `confirm(...)`.

###### .on('page', function(type="error", message, stack))
监听顶层页面错误。 这将在页面上抛出错误时触发。

###### .on('page', function(type="alert", message))
默认情况下，Nightmare会禁止window.alert弹出，但您仍然可以监听警报对话框的内容。

###### .on('page', function(type="prompt", message, response))
Nightmare默认情况下禁用window.prompt弹出，但你仍然可以监听到消息。如果您需要以不同的方式处理确认，则需要使用自己的预加载脚本。

###### .on('page', function(type="confirm", message, response))
Nightmare默认情况下会禁止window.confirm弹出，但你仍然可以监听到消息。如果您需要以不同的方式处理确认，则需要使用自己的预加载脚本。

###### .on('console', function(type [, arguments, ...]))
类型将是`log`，`warn`或`error`，参数是从控制台传递的。

##### Additional "console" events
监听`console.log(...)`，`console.warn(...)`和`console.error(...)`。

###### .on('console', function(type [, arguments, ...]))
类型将是`log`，`warn`或`error`，参数是从控制台传递的。

###### .on('console', function(type, errorMessage, errorStack))
如果在页面上使用了`console.log`，则触发此事件。但是如果注入的JavaScript代码（例如通过`.evaluate()`）使用`console.log`，则不会触发此事件。

##### .once(event, callback)
类似于`.on()`，但捕获页面事件与回调只执行一次。

##### .removeListener(event, callback)
删除事件的给定监听回调。

##### .screenshot([path][, clip])
获取当前页面的屏幕截图。用于调试。输出的总是一个`png`文件。这两个参数是可选的。如果提供了路径，它会将映像保存到磁盘。否则返回图像数据的`Buffer`。如果提供了`clip`（[如此处所述](https://github.com/electron/electron/blob/master/docs/api/browser-window.md#wincapturepagerect-callback)），则图像将剪切到矩形。

##### .html(path, saveType)
将当前页面作为html作为文件保存到给定路径的磁盘。 保存类型选项在[这里](https://github.com/atom/electron/blob/master/docs/api/web-contents.md#webcontentssavepagefullpath-savetype-callback)。

##### .pdf(path, options)
将PDF保存到指定的路径。选项在[这里](https://github.com/atom/electron/blob/v0.35.2/docs/api/web-contents.md#webcontentsprinttopdfoptions-callback)。

##### .title()
返回当前页面的标题。

##### .url()
返回当前页面的网址。


### Cookies

#### .cookies.get(name)
获取一个cookie的名称。 url将是当前的url。

#### .cookies.get(query)
使用查询对象查询多个Cookie。如果设置了query.name，它将返回它找到的第一个cookie，否则它将查询一个cookie数组。如果没有设置`query.url`，它将使用当前的url。例子：
```javascript
// get all google cookies that are secure
// and have the path `/query`
nightmare
  .goto('http://google.com')
  .cookies.get({
    path: '/query',
    secure: true
  })
  .then(function(cookies) {
    // do something with the cookies
  })
```
可用属性在此处记录：  
[https://github.com/atom/electron/blob/master/docs/api/session.md#sescookiesgetdetails-callback](https://github.com/atom/electron/blob/master/docs/api/session.md#sescookiesgetdetails-callback)

#### .cookies.get()
获取当前网址的所有Cookie。如果您想要获取所有网址的所有Cookie，请使用：`.get({url：null})`。

#### .cookies.set(name, value)
设置Cookie的`name`和`value`。最基本的形式，url将是当前的url。

#### .cookies.set(cookie)
设置cookie。如果没有设置`cookie.url`，它将在当前url上设置cookie。例子：
```javascript
nightmare
  .goto('http://google.com')
  .cookies.set({
    name: 'token',
    value: 'some token',
    path: '/query',
    secure: true
  })
  // ... other actions ...
  .then(function() {
    // ...
  })
```
可用属性在此处记录：   
[https://github.com/electron/electron/blob/master/docs/api/session.md#sescookiessetdetails-callback](https://github.com/electron/electron/blob/master/docs/api/session.md#sescookiessetdetails-callback)

#### .cookies.set(cookies)
一次设置多个Cookie。`cookies`是一个`cookie`对象的数组。看看上面的`.cookies.set(cookie)`文档，以更好地了解什么`cookie`应该是什么样子。

#### .cookies.clear([name])
清除所有网域的所有Cookie。
```javascript
nightmare
  .goto('http://google.com')
  .cookies.clearAll()
  // ... other actions ...
  .then(function() {
    //...
  });
```


### Proxies

Nightmare通过[switch](#switches)支持代理。  
如果您的代理需要[身份验证](#authenticationuser-password)，则还需要身份验证调用。  
以下示例不仅演示如何使用代理，您还可以运行它来测试您的代理连接是否正常工作：
```javascript
var Nightmare = require('nightmare');

var proxyNightmare = Nightmare({
  switches: {
    'proxy-server': 'my_proxy_server.example.com:8080' // set the proxy server here ...
  },
  show: true
});

proxyNightmare
  .authentication('proxyUsername', 'proxyPassword') // ... and authenticate here before `goto`
  .goto('http://www.ipchicken.com')
  .evaluate(function() {
    return document.querySelector('b').innerText.replace(/[^\d\.]/g, '');
  })
  .end()
  .then(function(ip) { // This will log the Proxy's IP
    console.log('proxy IP:', ip);
  });

// The rest is just normal Nightmare to get your local IP
var regularNightmare = Nightmare({ show: true });

regularNightmare
  .goto('http://www.ipchicken.com')
  .evaluate(function() {
    return document.querySelector('b').innerText.replace(/[^\d\.]/g, '');
  })
  .end()
  .then(function(ip) { // This will log the your local IP
    console.log('local IP:', ip);
  });
```


### Extending Nightmare

#### Nightmare.action(name, [electronAction|electronNamespace], action|namespace)
你可以添加自己的自定义动作到Nightmare的原型。例子：
```javascript
Nightmare.action('size', function (done) {
  this.evaluate_now(function() {
    var w = Math.max(document.documentElement.clientWidth, window.innerWidth || 0)
    var h = Math.max(document.documentElement.clientHeight, window.innerHeight || 0)
    return {
      height: h,
      width: w
    }
  }, done)
})

Nightmare()
  .goto('http://cnn.com')
  .size()
  .then(function(size) {
    //... do something with the size information
  });
```
> 记住，这是附加到static class `Nightmare`，而不是实例。
你会注意到我们使用了一个内部函数`evaluate_now`。这个函数不同于`nightmare.evaluate`，因为它立即运行，然而`nightmare.evaluate`需要在队列中等待。   
简单记住：当有疑问时，使用`evaluate`，如果您要创建自定义操作，请使用`evaluate_now`。技术原因是因为我们的动作已经进入到队列中，我们现在正在运行它，我们不应该让`evaluate`函数重新进入到队列中。   
我们还可以创建自定义命名空间。在内部我们像`nightmare.cookies.get`和`nightmare.cookies.set`这样做。如果您想要公开的操作包，这很有用，但它会使主要的Nightmare对象混乱。例如：

```javascript
Nightmare.action('style', {
  background: function (done) {
    this.evaluate_now(function () {
      return window.getComputedStyle(document.body, null).backgroundColor
    }, done)
  }
})

nightmare()
  .goto('http://google.com')
  .style.background()
  .then(function(background) {
    // ... do something interesting with background  
  })
```

您还可以添加自定义Electron动作。附加的Electron操作或命名空间操作采用`name`，`options`，`parent`， `win`，`renderer`，和 `done`。Electron动作先执行，镜像如何`.evaluate()`工作。例如：

> You can also add custom Electron actions. The additional Electron action or namespace actions take `name`, `options`, `parent`, `win`, `renderer`, and `done`. Note the Electron action comes first, mirroring how `.evaluate()` works. For example:

```javascript
Nightmare.action('clearCache',
  function(name, options, parent, win, renderer, done) {
    parent.respondTo('clearCache', function(done) {
      win.webContents.session.clearCache(done);
    });
    done();
  },
  function(message, done) {
    this.child.call('clearCache', done);
  });

Nightmare()
  .clearCache()
  .goto('http://example.org')
  //... more actions ...
  .then(function() {
    // ...
  });
```
...在导航到example.org之前，将清除浏览器的缓存。

#### .use(plugin)
`nightmare.use`对于在实例上重用一组任务很有用。查看[nightmare-swiftly ](https://github.com/segmentio/nightmare-swiftly)的一些例子。

#### Custom preload script
如果你需要做一些自定义的事情，当你第一次加载窗口环境，您可以指定自定义预加载脚本。这展示了如何这么做：
```javascript
const path = require('path');

var nightmare = Nightmare({
  webPreferences: {
    preload: path.resolve("custom-script.js")
    //alternative: preload: "absolute/path/to/custom-script.js"
  }
})
```
该脚本的唯一要求是，您需要提前这么做：
```javascript
window.__nightmare = {};
__nightmare.ipc = require('electron').ipcRenderer;
```
为了惠及于所有的Nightmare的来自浏览器的反馈，你可以改为复制Nightmare的[preload脚本](https://github.com/segmentio/nightmare/blob/master/lib/preload.js)的内容。





## Usage

#### Installation
Nightmare是一个Node.js的模块，所以它需要已经安装Node.js的环境。你仅仅需要用`npm install`来安装模块：
```
$ npm install --save nightmare
```

#### Execution

Nightmare是一个Node的模块，可以在Node.js脚本或模块中使用。这里有一个简单的脚本来打开网页：
```javascript
var Nightmare = require('nightmare'),
  nightmare = Nightmare();

nightmare.goto('http://cnn.com')
  .evaluate(function(){
    return document.title;
  })
  .end()
  .then(function(title){
    console.log(title);
  })
```
如果你将这个脚本储存成`cnn.js`，你可以在命令行像这样运行：
```
npm install nightmare
node cnn.js
```

#### Debugging
有三种好的方法可以获得更多信息：Headless浏览器发生了什么。
1.使用下面描述的`DEBUG = *`标志。
2.传递`{show：true}`到[Nightmare的构造函数](#nightmareoptions)，让它创建一个可见的，呈现的窗口，你可以看到发生了什么。
3.监听[特定事件](#onevent-callback)。   
   
要使用调试输出运行同一个文件，像`DEBUG=nightmare node cnn.js`这样运行，(在 Windows像`DEBUG=nightmare & node cnn.js`这样运行)。  
这将打印出一些关于发生了什么的额外信息：  
```
nightmare queueing action "goto" +0ms
nightmare queueing action "evaluate" +4ms
Breaking News, U.S., World, Weather, Entertainment & Video News - CNN.com
```

##### Debug Flags

所有的nightmare的消息  
`DEBUG=nightmare*`  
只有actions  
`DEBUG=nightmare:actions*`  
只有log  
`DEBUG=nightmare:log*`

#### Tests
Nightmare本身的自动测试使用[Mocha](http://mochajs.org/)和Chai，这两个都将通过`npm install`安装。要运行Nightmare的测试，只需运行`make test`。  
当测试完成后，你会看到这样：

```
make test
  ․․․․․․․․․․․․․․․․․․
  18 passing (1m)
```
注意，如果你正在使用`xvfb`，`make test`，将在`xvfb-run`装饰者模式下自动运行测试。如果您计划运行测试而不首先运行`xvfb`，将`HEADLESS`环境变量设置为`0`。

---

```
                                 ########################
                     ################################################
                  ######################################################
               ###############################################################
            #####################################################################
         ###########################################################################
      ##############################################################################
      #################################################################################
   ####################################################################################
   #######################################################################################
      ##############################################################################fff###
EEE   ####################################################################################
#######################################################################################jjj
##########################################################################################
fff,,,####################################################################################
      EEE###,,,   WWW###################################################      ######   ###
   WWW      ######      #############################################   ######         LLL
   ###      ############   ####################################      ############
   ###   ###   ############   ##############################   ############   ###   fff
      #########            ######      ############:::   ######            ttt######
      ######                     ###############   ######GGG                  ######:::
   ttt######                              ######                              WWW######
   ###...###                           GGG#########                           #########
         ###                           ######LLL######                        ###
         ###...                     ######      ######                        ###
      #########               ############         #########               ######
      #################################            #################################DDD
   KKK#################################            ####################################
   ###         ########################               #####################         ###
                                 ######   ######   #########
                                 ###########################
                              ###############   ;;;fff
                           ###                        #########
                           ###EEEDDD      ######      KKKEEE###
                           ######EEE###         ######      fff
                           ###jjj######   ###   ###
                                 jjj   ############
                                                ;;;
```
