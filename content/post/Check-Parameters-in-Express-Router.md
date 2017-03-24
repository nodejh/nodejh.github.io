+++
date = "2016-10-15T06:46:05+08:00"
description = "检测 Express 路由中的参数合法性"
title = "Check Parameters in Express Router"
tags = ["Node.js", "JavaScript", "Express"]

+++

本文以 Express 框架为基础，讲诉如何通过一个中间件来检测 Express 路由中传输的参数是否合法。

几乎对于任何应用，前后端都需要进行传输数据。不管是通过 HTTP 请求的 POST 方法还是 GET 方法，数据校验都是必要的操作。

对于大部分 API 来说，可能只需要判断传入的参数是否为 undefined 或 null，所以这个时候，为了减少重复代码，我们可以写一个简单的中间件来处理路由中的参数。

这个中间件的需求如下：

+ 检测路由中的一般参数是否为 undefined、null、[]、''
+ 中间件同时还需要能对特殊参数做处理，如一个参数值在 1-100 之间

最终写出来的处理参数的模块：

```
/**
 * 检测路由中的参数
 * @method checkParameters
 * @param  {object}        req     http请求
 * @param  {object}        res     http响应
 * @param  {Function}      next    
 * @param  {[type]}        special 特殊参数，
 *                                 格式： [{eval: 'req.query.id>0', key:'id', type: 'query'}]
 *                                 key 属性的值必须和eval中对应，且type只能是 query/params/body
 * @return {object}        若参数错误，直接返回json到前端；正确，则next()
 */
function checkParameters(req, res, next, special) {
  // console.log('params: ', req.params);
  // console.log('body: ', req.body);
  // console.log('query: ', req.query);
  console.log('specialParams: ', typeof special);
  // 判断是否传入第四个参数
  if (Array.isArray(special)) {
    // 判断特殊参数
    for (let i = 0; i < special.length; i++) {
      // console.log('special[i]: ', special[i]);
      // console.log('special[i] eval: ', special[i].hasOwnProperty('eval'));
      // console.log('special[i] key: ', special[i].hasOwnProperty('key'));
      // console.log('special[i] type: ', special[i].hasOwnProperty('type'));

      // 判断是否具有 eval, key, type 三个参数
      if (! (special[i].hasOwnProperty('eval')
        && special[i].hasOwnProperty('key')
        && special[i].hasOwnProperty('type'))) {
        return res.json({
          code: 90001,
          msg: '检测参数属否正确时，传入的特殊参数格式不完全',
          detail: special
        });
      }
      // 判断 eval, key, type 三个参数是否为 undefined
      if ( !(typeof special[i]['eval'] !== undefined
      && typeof special[i]['key'] !== undefined
      && typeof special[i]['type'] !== undefined)) {
        return res.json({
          code: 90002,
          msg: '检测参数属否正确时，传入的特殊参数为 undefined',
          detail: special
        });
      }
      const evalString = special[i]['eval'];
      const type = special[i]['type'];
      const key = special[i]['key'];
      // 判断 key 和 eval 是否匹配
      console.log('length: ', evalString
      .split('req.' + type + '.' + key)
      .length);
      const length = evalString.split('req.' + type + '.' + key).length;
      if (length < 1) {
        return res.json({
          code: 90003,
          msg: '检测参数属否正确时，传入的特殊参数为格式不正确',
          detail: special
        });
      }
      // 执行 eval
      if (!eval(evalString)) {
        return res.json({
          code: 90004,
          msg: '检测参数属否正确时，参数不匹配传入的条件 ' + evalString,
          detail: special,
          parameters: req[type]
        });
      }
      // 从普通参数中删除特殊参数
      // console.log('delete: ', req[type][key]);
      delete req[type][key];
    }
  }

  const params = req.params;
  // 去掉 req.params 中 {'0': '', ...} 这一项
  delete params['0'];
  // console.log('params: ', params);
  const body = req.body;
  const query = req.query;
  const common = Object.assign(params, body, query);
  console.log('common: ', common);
  // 检测参数属否为undefined
  for (let i in common) {
    if (typeof common[i] === 'undefined') {
      return res.json({
        code: 90005,
        msg: '检测参数属否正确时，' + i + ' 参数为undefined',
        detail: common
      });
    }
    // 检测参数属否为null
    if (common[i] === null) {
      return res.json({
        code: 90006,
        msg: '检测参数属否正确时，' + i + ' 参数为null',
        detail: common
      });
    }
    // 检测参数是否为空字符串 ''
    if (common[i] === '') {
      return res.json({
        code: 90007,
        msg: '检测参数属否正确时，' + i + ' 参数为空字符串',
        detail: common
      });
    }
    // 检测参数是否为空数组 []
    if (common[i] === '') {
      return res.json({
        code: 90008,
        msg: '检测参数属否正确时，' + i + ' 参数为空数组',
        detail: common
      });
    }
  }

  return next();
}

module.exports = checkParameters;

```

#### 函数详解

**1. 特殊参数的处理**

这个模块，首先对特殊参数做处理。`special` 这个参数是一个数组，格式如下：

```
[{
  eval: 'parseInt(req.query.id) > 1 && parseInt(req.query.id) < 10',
  key: 'id',
  type: 'query'
}, {
  eval: 'parseInt(req.query.age) === 11',
  key: 'age',
  type: 'query'
}]
```

key 属性的值必须和eval中对应，且type只能是 query/params/body。

+ 首先判断函数是否传入第四个参数，并且第四个参数是否为数组，如果这两个条件成立，说明有特殊参数需要处理。
+ 然后判断是否具有 eval, key, type 三个参数。eval 参数是一个参数条件表达式，如 'id>0'，最终通过 `eval()` 函数来执行这个表达式。`key` 表示参数属性名，type 表示是通过何种方式传递的参数，如 query/params/body。
+ 接下来判断  eval, key, type 三个参数自身是否为 undefined 或 null
+ 再然后判断 判断 key 和 eval 是否匹配。key 和 eval 的匹配，主要是为了后面从一般参数中删除特殊参数
+ 再执行 eval 表达式，判断表达式是否成立，即特殊参数值是否合法
+ 如果以上都为真，说明特殊参数合法，最后则使用 delete 从参数中删除特殊参数

**2. 普通参数处理**

处理完毕特殊参数后，就需要处理普通参数。普通参数只需要满足：

+ 不为 `undefined`
+ 不为 `null`
+ 不为空字符串 `''`
+ 不为空数组 `[]`

普通参数，即 `req.body` `req.params` `req.query` 中的所有非特殊参数。

除了在判断特殊参数的时候，需要 delete 特殊参数，还需要 `delete req.params['0']`。因为对于 `req.params`，当 URL 没有 `path` 部分的时候，`req.params['0']` 为 `''`。

处理完毕之后，若所有参数都合法，则返回 `next()`。


#### 模块使用


**1. 创建没有挂载路径的中间件**

首先在 Express 项目入口文件，创建一个参数处理中间件：

```
// app.js
const checkParameters = require('./checkParameters');
// ...
app.use(function(req, res, next) {
  checkParameters(req, res, next);
});
```

注意这里的 `app.use()` 需要放在创建具体路由中间件如`app.use('/', index)` 之前。

当 Express 接收到 HTTP 请求后，首先该中间件会检测参数是否合法（只判断是否为 null 或 undefine 或 '' 或 []）。如果不合法，则直接返回；如果合法，则程序继续执行，由匹配该请求路径的路由继续完成该请求。


**2. 创建挂载路径的中间件**

如果需要对某个路由的特殊参数做处理，则需要创建一个挂载路径的中间件。

比如为们要判断 `/index` 路由的 GET 请求中的 `id` 参数，其中 `id` 的取值范围是 1-100，URL 形式如 `http://localhost:3000/index?id=10`，则可以这样使用该模块：

```
// index.js
const checkParameters = require('./checkParameters');

// 挂载路径的中间件
router.get('/index', function(req, res, next) {
  const special = {
    eval: 'parseInt(req.query.id) >= 1 && parseInt(req.query.id) <= 100',
    key: 'id',
    type: 'query'
  };
  checkParameters(req, res, next, special);
});
// 正常逻辑处理
router.get('/index', function(req, res, next) {
  // 正常逻辑
});
```

当 Express 接收到一个 HTTP 请求的时候，首先还是没有挂载路径的中间件先处理请求，检测参数；然后该挂载路径的中间件再检测参数；最后才执行正常处理逻辑。

---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/5](https://github.com/nodejh/nodejh.github.io/issues/5)
