+++
title = "Webstorm: Node.js core module source not configured"
description = "Webstorm 中 Node.js 核心模块配置"
date = "2016-04-26T23:37:35+08:00"
tags = ["Webstorm"]

+++


在 Webstrom 中引入 Node.js 自带模块的时候，Webstorm 有如下错误提示：

```
Node.js core module source not configured
```

这是因为没有配置 Webstorm 支持 Node.js 自带模块。

<!--more-->

打开 `File | Settings | Languages & Frameworks | Node.js and NPM`，在右侧面板会发现 `Node.js Core library is not enabled.`，点击旁边的 `Enable` 按钮然后再点击下面的 `OK` 按钮即可。


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/22](https://github.com/nodejh/nodejh.github.io/issues/22)
