+++
title = "删除GIT中的.DS_Store"
description = "How to Remove .DS_Store on macOS"
date = "2016-02-24T14:25:25+08:00"
tags = ["Mac OX","GIT",".DS_Store"]

+++


<!--more-->

### .DS_Store 是什么

使用 Mac 的用户可能会注意到，系统经常会自动在每个目录生成一个隐藏的 `.DS_Store 文件`。`.DS_Store` (英文全称 Desktop Services Store)是一种由苹果公司的Mac OS X操作系统所创造的隐藏文件，目的在于存贮目录的自定义属性，例如文件们的图标位置或者是背景色的选择。相当于 Windows 下的 `desktop.ini`。


### 删除 .DS_Store

如果你的项目中还没有自动生成的 `.DS_Store` 文件，那么直接将 `.DS_Store` 加入到 `.gitignore` 文件就可以了。如果你的项目中已经存在 `.DS_Store` 文件，那就需要先从项目中将其删除，再将它加入到 `.gitignore`。如下：

```
# 删除项目中的所有.DS_Store。这会跳过不在项目中的 .DS_Store
find . -name .DS_Store -print0 | xargs -0 git rm -f --ignore-unmatch
# 将 .DS_Store 加入到 .gitignore
echo .DS_Store >> ~/.gitignore
# 更新项目
git add --all
git commit -m '.DS_Store banished!'
```

如果你只需要删除磁盘上的 `.DS_Store`，可以使用下面的命令来删除当前目录及其子目录下的所有` .DS_Store` 文件:

```
find . -name '*.DS_Store' -type f -delete
```

### 禁用或启用自动生成

+ 禁止.DS_store生成：

```
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

+ 恢复.DS_store生成：

```
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/18](https://github.com/nodejh/nodejh.github.io/issues/18)
