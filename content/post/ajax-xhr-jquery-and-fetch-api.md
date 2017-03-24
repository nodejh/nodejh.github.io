+++
title = "AJAX: XHR, jQuery and Fetch API"
description = "分别使用 XHR、jQuery 和 Fetch 实现 AJAX"
date = "2016-09-26T19:58:03+08:00"
tags = ["JavaScript"]

+++


本文详细讲述如何使用原生 JS、jQuery 和 Fetch 来实现 AJAX。

AJAX 即 Asynchronous JavaScript and XML，异步的 JavaScript 和 XML。使用 AJAX 可以无刷新地向服务端发送请求接收服务端响应，并更新页面。

<!--more-->

## 一、原生 JS 实现 AJAX

JS 实现 AJAX 主要基于浏览器提供的 XMLHttpRequest（XHR）类，所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。


#### 1. 获取XMLHttpRequest对象

```
// 获取XMLHttpRequest对象
var xhr = new XMLHttpRequest();
```

如果需要兼容老版本的 IE (IE5, IE6) 浏览器，则可以使用 ActiveX 对象：

```
var xhr;
if (window.XMLHttpRequest) { // Mozilla, Safari...
  xhr = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE
  try {
    xhr = new ActiveXObject('Msxml2.XMLHTTP');
  } catch (e) {
    try {
      xhr = new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) {}
  }
}
```

#### 2. 发送一个 HTTP 请求

接下来，我们需要打开一个URL，然后发送这个请求。分别要用到 XMLHttpRequest 的 open() 方法和 send() 方法。

```
// GET
var xhr;
if (window.XMLHttpRequest) { // Mozilla, Safari...
  xhr = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE
  try {
    xhr = new ActiveXObject('Msxml2.XMLHTTP');
  } catch (e) {
    try {
      xhr = new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) {}
  }
}
if (xhr) {
  xhr.open('GET', '/api?username=admin&password=root', true);
  xhr.send(null);
}
```

```
// POST
var xhr;
if (window.XMLHttpRequest) { // Mozilla, Safari...
  xhr = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE
  try {
    xhr = new ActiveXObject('Msxml2.XMLHTTP');
  } catch (e) {
    try {
      xhr = new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) {}
  }
}
if (xhr) {
  xhr.open('POST', '/api', true);
  // 设置 Content-Type 为 application/x-www-form-urlencoded
  // 以表单的形式传递数据
  xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
  xhr.send('username=admin&password=root');
}
```

`open()` 方法有三个参数：

+ `open()` 的第一个参数是 HTTP 请求方式 – GET，POST，HEAD 或任何服务器所支持的您想调用的方式。按照HTTP规范，该参数要大写；否则，某些浏览器(如Firefox)可能无法处理请求。有关HTTP请求方法的详细信息可参考 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
+ 第二个参数是请求页面的 URL。由于同源策略（Same origin policy）该页面不能为第三方域名的页面。同时一定要保证在所有的页面中都使用准确的域名，否则调用 `open()` 会得到 `permission denied` 的错误提示。
+ 第三个参数设置请求是否为异步模式。如果是 `TRUE`，JavaScript 函数将继续执行，而不等待服务器响应。这就是 AJAX 中的 A。

如果第一个参数是 `GET`，则可以直接将参数放在 url 后面，如：`http://nodejh.com/api?name=admint&password=root`。

如果第一个参数是 `POST`，则需要将参数写在 send() 方法里面。send() 方法的参数可以是任何想送给服务器的数据。这时数据要以字符串的形式送给服务器，如：`name=admint&password=root`。或者也可以传递 JSON 格式的数据：

```
// 设置 Content-Type 为 application/json
xhr.setRequestHeader('Content-Type', 'application/json');
// 传递 JSON 字符串
xhr.send(JSON.stringify({ username:'admin', password:'root' }));
```

如果不设置请求头，原生 AJAX 会默认使用 Content-Type 是 `text/plain;charset=UTF-8` 的方式发送数据。

关于 Content-Type 更详细的内容，将在以后的文章中解释说明。


#### 3. 处理服务器的响应

当发送请求时，我们需要指定如何处理服务器的响应，我们需要用到 onreadystatechange 属性来检测服务器的响应状态。使用 onreadystatechange 有两种方式，一是直接 onreadystatechange 属性指定一个可调用的函数名，二是使用一个匿名函数：

```
// 方法一 指定可调用的函数
xhr.onreadystatechange = onReadyStateChange;
function onReadyStateChange() {
  // do something
}

// 方法二 使用匿名函数
xhr.onreadystatechange = function(){
    // do the thing
};
```

接下来我们需要在内部利用 readyState 属性来获取当前的状态，当 readyState 的值为 4，就意味着一个完整的服务器响应已经收到了，接下来就可以处理该响应：

```
// readyState的取值如下
// 0 (未初始化)
// 1 (正在装载)
// 2 (装载完毕)
// 3 (交互中)
// 4 (完成)
if (xhr.readyState === 4) {
    // everything is good, the response is received
} else {
    // still not ready
}
```

完整代码如下：

```
// POST
var xhr;
if (window.XMLHttpRequest) { // Mozilla, Safari...
  xhr = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE
  try {
    xhr = new ActiveXObject('Msxml2.XMLHTTP');
  } catch (e) {
    try {
      xhr = new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) {}
  }
}
if (xhr) {
  xhr.onreadystatechange = onReadyStateChange;
  xhr.open('POST', '/api', true);
  // 设置 Content-Type 为 application/x-www-form-urlencoded
  // 以表单的形式传递数据
  xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
  xhr.send('username=admin&password=root');
}


// onreadystatechange 方法
function onReadyStateChange() {
  // 该函数会被调用四次
  console.log(xhr.readyState);
  if (xhr.readyState === 4) {
    // everything is good, the response is received
    if (xhr.status === 200) {
      console.log(xhr.responseText);
    } else {
      console.log('There was a problem with the request.');
    }
  } else {
    // still not ready
    console.log('still not ready...');
  }
}
```

当然我们可以用onload来代替onreadystatechange等于4的情况，因为onload只在状态为4的时候才被调用，代码如下：

```
xhr.onload = function () {    // 调用onload
    if (xhr.status === 200) {    // status为200表示请求成功
        console.log('执行成功');
    } else {
        console.log('执行出错');
    }   
}
```

然而需要注意的是，IE对 onload 属性的支持并不友好。除了 onload 还有以下几个属性也可以用来监测响应状态：

+ onloadstart
+ onprogress
+ onabort
+ ontimeout
+ onerror
+ onloadend


## 二、 jQuery 实现 AJAX

jQuery 作为一个使用人数最多的库，其 AJAX 很好的封装了原生 AJAX 的代码，在兼容性和易用性方面都做了很大的提高，让 AJAX 的调用变得非常简单。下面便是一段简单的 jQuery 的 AJAX 代码：

```
$.ajax({
  method: 'POST',
  url: '/api',
  data: { username: 'admin', password: 'root' }
})
  .done(function(msg) {
    alert( 'Data Saved: ' + msg );
  });
```

对比原生 AJAX 的实现，使用 jQuery 就异常简单了。当然我们平时用的最多的，是下面两种更简单的方式：

```
// GET
$.get('/api', function(res) {
  // do something
});

// POST
var data = {
  username: 'admin',
  password: 'root'
};
$.post('/api', data, function(res) {
  // do something
});
```

## 三、Fetch API

使用 jQuery 虽然可以大大简化 XMLHttpRequest 的使用，但 XMLHttpRequest 本质上但并不是一个设计优良的 API：
+ 不符合关注分离（Separation of Concerns）的原则
+ 配置和调用方式非常混乱
+ 使用事件机制来跟踪状态变化
+ 基于事件的异步模型没有现代的 Promise，generator/yield，async/await 友好

Fetch API 旨在修正上述缺陷，它提供了与 HTTP 语义相同的 JS 语法，简单来说，它引入了 `fetch()` 这个实用的方法来获取网络资源。

Fetch 的浏览器兼容图如下：

![ajax-js-jquery-and-fetch-api-0.png](/images/ajax-js-jquery-and-fetch-api-0.png)

原生支持率并不高，幸运的是，引入下面这些 polyfill 后可以完美支持 IE8+：

+ 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: [es5-shim, es5-sham](https://github.com/es-shims/es5-shim)
+ 引入 Promise 的 polyfill: [es6-promise](https://github.com/stefanpenner/es6-promise)
+ 引入 fetch 探测库：[fetch-detector](https://github.com/camsong/fetch-detector)
+ 引入 fetch 的 polyfill: [fetch-ie8](https://github.com/camsong/fetch-ie8)
+ 可选：如果你还使用了 jsonp，引入 [fetch-jsonp](https://github.com/camsong/fetch-jsonp)
+ 可选：开启 Babel 的 runtime 模式，现在就使用 async/await

#### 1. 一个使用 Fetch 的例子

先看一个简单的 Fetch API 的例子 🌰 ：

```
fetch('/api').then(function(response) {
  return response.json();
}).then(function(data) {
  console.log(data);
}).catch(function(error) {
  console.log('Oops, error: ', error);
});
```

使用 ES6 的箭头函数后：

```
fetch('/api').then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.log('Oops, error: ', error))
```

可以看出使用Fetch后我们的代码更加简洁和语义化，链式调用的方式也使其更加流畅和清晰。但这种基于 Promise 的写法还是有 Callback 的影子，我们还可以用 `async/await` 来做最终优化：

```
async function() {
  try {
    let response = await fetch(url);
    let data = response.json();
    console.log(data);
  } catch (error) {
    console.log('Oops, error: ', error);
  }
}
```

使用 `await` 后，写代码就更跟同步代码一样。`await` 后面可以跟 Promise 对象，表示等待 Promise `resolve()` 才会继续向下执行，如果 Promise 被 `reject()` 或抛出异常则会被外面的 `try...catch` 捕获。

Promise，generator/yield，await/async 都是现在和未来 JS 解决异步的标准做法，可以完美搭配使用。这也是使用标准 Promise 一大好处。

#### 2. 使用 Fetch 的注意事项

+ Fetch 请求默认是不带 cookie，需要设置 `fetch(url, {credentials: 'include'})``
+ 服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject



接下来将上面基于 XMLHttpRequest 的 AJAX 用 Fetch 改写：

```
var options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ username: 'admin', password: 'root' }),
    credentials: 'include'
  };

fetch('/api', options).then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.log('Oops, error: ', error))
```
---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/15](https://github.com/nodejh/nodejh.github.io/issues/15)


