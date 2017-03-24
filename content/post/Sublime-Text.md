+++
categories = ["y"]
date = "2017-02-22T14:12:28+08:00"
description = ""
draft = true
tags = ["x"]
title = "Sublime Text"

+++


## auto save


+ Mac - click Sublime Text 2 > Preferences > Settings - User
+ Windows - click Preferences > Settings - User

Add this line

```
{ "save_on_focus_lost": true }
```

##  CSS排序

CSS属性的顺序一般不重要，因为无论何种顺序浏览器都能正确渲染。但排序所有的属性还是有助于代码的整洁。在Sublime Text中，选中CSS属性后按F5就可以按字母顺序排序。

![css排序](http://segmentfault.com/img/bVchy6)

也可以使用 CSSComb 等第三方插件，更详细的控制排序的方法。

[12个不可不知的Sublime Text应用技巧和诀窍](https://segmentfault.com/a/1190000000505218#articleHeader0)


## 插件

已安装

sidebarenhancement（侧边栏增强，在侧边右键之后多了一些功能，挺实用的）
docblockr（文档注释，输入/*之后按一下tab就可以生成文档注释）
Bracket Highlighter（匹配括号高亮，自带的感觉高亮不强）

未安装

QuoteHTML，把HTML拼接成js插入字符串，神器
FileHeader，文件模板 , 可自动更新修改时间
AutoPrefixer，浏览器私有属性前缀补全 (Node.js依赖)
Better Completion，全能代码提示
LiveStyle，双向更改无刷新实时预览 , 包含chrome插件 Emmet LiveStyle
PackageResourceViewer，插件修改必备，ctrl+shift+p 调用 Open Resource

## 配置

```
{
	"ignored_packages":
	[
		"Vintage"
	],
	"save_on_focus_lost": true,
	"spell_check": false,
	"highlight_line": true,
	// 移除行尾的空格
	"trim_trailing_white_space_on_save": true,
	// 拖尾换行
	"ensure_newline_at_eof_on_save": true,
	// 字体
	"font_face": "Consolas",
	"tab_size": 2,
}
```




