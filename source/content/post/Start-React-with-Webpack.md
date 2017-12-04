+++
title = "Webpack 和 React 实战"
date = "2016-09-09T04:11:13+08:00"
tags = ["React.js", "Webpack"]
description = "Start React with Webpack"

+++

## TL;DR

```
$ git clone https://github.com/nodejh/start-react-with-webpack react-sample
$ cd react-sample && npm install
$ npm run dev
```

<!--more-->


然后打开浏览器输入 `http://localhost:8080`，并尝试随意修改一下 app 目录里面的代码，就能看到效果了。

为了避免包版本问题导致程序不能运行，根目录下有一个 `npm-shrinkwrap.json` 文件，这里面所有包的版本都是固定的。 `npm install` 时首先会检查在根目录下有没有 `npm-shrinkwrap.json`，如果 shrinkwrap 文件存在的话，npm 会使用它（而不是 `package.json`）来确定安装的各个包的版本号信息。

## 1. 安装并配置 Webpack

首先创建并初始化一个项目目录：

```
$ mkdir react-sample && cd react-sample
$ npm init
```

安装 `webpack`：

```
$ npm i webpack --save-dev
```

然后配置 `webpack.config.js`：

```
# 创建一个 webpack.config.js 文件
$ touch webpack.config.js
```
<!-- more -->

在该文件中加入下面的内容：

```
const webpack = require('webpack');
const path = require('path');

// 定义打包目录路径
const BUILD_DIR = path.resolve(__dirname, './build');
// 定义组件目录路径
const APP_DIR = path.resolve(__dirname, './app');

const config = {
  entry: `${APP_DIR}/index.jsx`, // 文件打包的入口点
  output: {
    path: BUILD_DIR, // 输出目录的绝对路径
    filename: 'bundle.js', // 输出的每个包的相对路径
  },
  resolve: {
    extensions: ['', '.js', '.jsx'], // 开启后缀名的自动补全
  },
};

module.exports = config;
```

这是一个最基本的 webpack 配置文件。

接下来在 `build/` 目录中创建一个 `index.html` 文件：

```
<html>
  <head>
    <meta charset="utf-8">
    <title>Start React with Webpack</title>
  </head>
  <body>
    <div id="app" />
    <script type="text/javascript" src="./bundle.js"></script>
  </body>
</html>
```


## 2. 配置加载器 babel-loader

加载器是把一个资源文件作为入参转换为另一个资源文件的 node.js 函数。

由于我们写 React 的时候使用的是 JSX 语法和 ES6 语法，而浏览器并不完全支持它们。所以需要使用 [`babel-loader`](https://github.com/babel/babel-loader) 来让 webpack 加载 JSX 和 ES6 的文件。

`babel-loader` 的主要作用如下图：

![Babel](/images/Start-React-with-Webpack-babel-loader.png)

安装依赖包：

```
$ npm i babel-core babel-loader babel-preset-es2015 babel-preset-react --save-dev
```

`babel-preset-es2015` 是转换 `ES6` 的包；`babel-preset-react` 是转换 JSX 的包。

接下来需要修改 `webpack.config.js`：

```
// Existing Code ....
const config = {
  // Existing Code ....
  module: {
    loaders: [{
      test: /\.(js|jsx)$/,
      exclude: /(node_modules|bower_components)/,
      loader: 'babel-loader',
      query: {
        presets: ['es2015', 'react']
      }
    }]
  }
};
```


## 3. Hello React

安装 React：

```
$ npm i react react-dom --save
```

在 `app` 目录下新建一个 `index.jsx` 文件，然后将下面的内容添加到 `index.jsx` 中：

```
import React from 'react';
import {render} from 'react-dom';

class App extends React.Component {
  render () {
    return <h1> Hello React!</h1>;
  }
}

render(<App/>, document.getElementById('app'));

```

这个时候，执行下面的命令打包：

```
webpack -w
```

`-w` 参数表示持续监测项目目录，如果文件发生修改，则重新打包。

打包完成后，将 `build/index.html` 用浏览器打开，就能看到 `Hello React!`，如下：

![hello_world.png](/images/Start-React-with-Webpack-hello_world.png)



## 4. 自动刷新和热加载

懒是第一生产力。每次写完代码，都要重新打包，重新刷新浏览器才能看到结果，显然很麻烦。

那有没有能够自动刷新浏览器的方法呢？当然有，这个时候就需要 webpack-dev-server 这个包。

```
$ npm install webpack-dev-server -g
```

`webpack-dev-server` 提供了两种自动刷新模式：

**Iframe 模式**

+ 不需要额外配置，只用修改路径
+ 应用被嵌入了一个 iframe 内部，页面顶部可以展示打包进度信息
+ 因为 Iframe 的关系，如果应用有多个页面，无法看到当前页面的 URL 信息

**inline 模式**

+ 需要添加 --inline 配置参数
+ 提示信息在控制台中和浏览器的console中显示
+ 页面的 URL 改变，可以在浏览器地址栏看见

接下来启动 webpack-dev-server：

```
$ webpack-dev-server --inline --hot --content-base ./build/
```

`--hot` 参数就是热加载，即在不刷新浏览器的条件下，应用最新的代码更新。在浏览器中可能看到这样的输出：

```
[HMR] Waiting for update signal from WDS...
[WDS] Hot Module Replacement enabled.
```

`--content-base ./` 参数表示将当前目录作为 server 根目录。命令启动后，会在 8080 端口创建一个 HTTP 服务，通过访问 `http://localhost:8080/index.html` 就可以访问我们的项目了，并且修改了项目中的代码后，浏览器会自动刷新并实现热加载。

当然，命令行输入这么长，还是不太方便，所以还有一种更简单的方式，在 `package.json` 中配置 webpack develop server：

```
// Existing Code ....
"scripts": {
    "dev": "webpack-dev-server --inline --hot --content-base ./build/"
  }
```

然后通过 `npm start dev` 来启动即可。


## 5. 添加一个新的组件

在 `app` 目录中新建一个 `AwesomeComponent.jsx` 文件，并添加如下代码：

```
import React, { Component } from 'react';

class AwesomeComponent extends Component {
  constructor(props) {
    super(props);
    this.state = {
      likesCount: 0
    };
    this.onLike = this.onLike.bind(this);
  }


  onLike() {
    let newLikesCount = this.state.likesCount + 1;
    this.setState({
      likesCount: newLikesCount
    });
  }


  render() {
    return (
      <div>
        Likes: <span>{this.state.likesCount}</span>
        <div>
          <button onClick={this.onLike}>Like Me</button>
        </div>
      </div>
    );
  }
}


export default AwesomeComponent;
```

然后修改 `index.jsx`：

```
// ...
import AwesomeComponent from './AwesomeComponent.jsx';
// ...
class App extends React.Component {
  render () {
    return (
      <div>
        <p> Hello React!</p>
        <AwesomeComponent />
      </div>
    );
  }
}

// ...
```

![like.png](/images/Start-React-with-Webpack-like.png)


---

## UPDATE

### 2016.10.15

+ 更新 webpack-dev-server 的配置方法

##### 设置 webpack-dev-server (old)

上面我们直接通过浏览器浏览的 `html` 文件，接下来我们需要利用 `webpack-dev-server` 来创建一个 HTTP Server。

首先安装 `webpack-dev-server`：

```
$ npm i webpack-dev-server --save-dev
```

然后在 `package.json` 的 `script` 里面加入 `build` 和 `dev` 两个命令：

```
{
  "scripts": {
    "build": "webpack",
    "dev": "webpack-dev-server --devtool eval --progress --colors --hot --content-base build"
  }
}
```

+ webpack-dev-server - 在 localhost:8080 建立一个 Web 服务器
+ --devtool eval - 为你的代码创建源地址。当有任何报错的时候可以让你更加精确地定位到文件和行号
+ --progress - 显示合并代码进度
+ --colors - 命令行中显示颜色！
+ --content-base build - 指向设置的输出目录

然后就可以使用 `npm run dev` 的命令来启动项目：

```
$ npm run dev
```

在浏览器地址栏输入 `localhost:8080` 即可看到页面。


如果需要浏览器自动刷新，将 `webpack.config.js` 中的 `entry: APP_DIR + '/index.jsx` 改为下面这样：

```
entry: [
   'webpack-dev-server/client?http://localhost:8080',
   'webpack/hot/dev-server',
    APP_DIR + '/index.jsx'
]
```

这样的话，每次当代码发生变化之后，webpack 会自动重新打包，浏览器也会自动刷新页面。



### 2016.11.19 更新

+ 使用 ES6 语法编写 webpack.config.js
+ 修改 babel-loader 加载器的配置方法：将添加 `.babelrc` 文件改为在 webpack.config.js 中配置
+ 🐛：webpack-dev-server --inline --hot --content-base ./build/ ➡️ webpack-dev-server --inline --hot --content-base ./build/

##### babel-loader 加载器的配置方法(old)

接下来需要配置 babel-loader，告诉 webpack 我们使用了 ES6 和 JSX 插件。先通过touch .babelrc 创建一个名为 .babelrc 的配置文件。然后加入下面的代码：

```
{
  "presets" : ["es2015", "react"]
}
```
然后再修改 webpack.config.js，使 webpack 在打包的时候，使用 babel-loader：

```
// Existing Code ....
var config = {
  // Existing Code ....
  module: {
    loaders : [
      {
        test : /\.jsx?/,
        include : APP_DIR,
        loader : 'babel'
      }
    ]
  }
}
```

##### 自动刷新和热加在的配置(old, wrong)

当然，命令行输入这么长，还是不太方便，所以还有一种更简单的方式，直接在 `webpack.cofig.js` 中配置 webpack develop server：

```
{
  entry: {
    // ...
  },
  // ...
  devServer: {
    hot: true,
    inline: true
  }
}
```

---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/12](https://github.com/nodejh/nodejh.github.io/issues/12)
