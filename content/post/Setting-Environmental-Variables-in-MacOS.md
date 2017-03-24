+++
categories = ["macOS"]
date = "2017-03-19T17:15:37+08:00"
description = "macOS/Linux 环境变量设置"
draft = false
tags = ["macOS", "SHELL"]
title = "Setting Environmental Variables in MacOS"

+++
我们安装一个软件后，之所以能够使用一些与该软件相关的命令，是因为该命令被添加到了系统的环境变量里面。比如安装完 Atom 之后，就可以使用 `atom` 命令打开文件。有时候我们需要自己设置环境变量，MacOS 设置环境变量有很多种方法，最常用的是编辑当前 SHELL 对应的用户级环境变量配置文件，如 `bash` 对应的 `.bash_profile`。

MacOS 和 Linux 都是类 Unix 系统，它们添加环境变量的方式也是类似的。本文以 macOS 为例。

## SEHLL 类型

在添加环境变量之前，首先要知道使用的是什么 SHELL。MacOS 内置了多种 SHELL，可通过 `cat /etc/shells` 查看：

```bash
$ cat /etc/shells
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
$ echo $SHELL
/bin/zsh
```

+ `sh`（全称 Bourne Shell）是UNIX最初使用的 shell，而且在每种 UNIX 上都可以使用。Bourne Shell 在 shell 编程方便相当优秀，但在处理与用户的交互方便作得不如其他几种 shell。
+ `bash`（全称 Bourne Again Shell）LinuxOS 默认的，它是 Bourne Shell 的扩展。与 Bourne Shell 完全兼容，并且在 Bourne Shell 的基础上增加了很多特性。可以提供命令补全，命令编辑和命令历史等功能。它还包含了很多 C Shell 和 Korn Shell 中的优点，有灵活和强大的编辑接口，同时又很友好的用户界面。
+ `csh`（全称 C Shell）是一种比 Bourne Shell更适合的变种 Shell，它的语法与 C 语言很相似。
+ `Tcsh` 是 Linux 提供的 C Shell 的一个扩展版本。
+ `Tcsh` 包括命令行编辑，可编程单词补全，拼写校正，历史命令替换，作业控制和类似 C 语言的语法，他不仅和 Bash Shell 提示符兼容，而且还提供比 Bash Shell 更多的提示符参数。
+ `ksh`（全称 Korn Shell）集合了 C Shell 和 Bourne Shell 的优点并且和 Bourne Shell 完全兼容。
+ `pdksh` 是 Linux 系统提供的 ksh 的扩展。pdksh 支持人物控制，可以在命令行上挂起，后台执行，唤醒或终止程序。
+ `zsh` Zsh 是一款功能强大终端（shell）软件，既可以作为一个交互式终端，也可以作为一个脚本解释器。它在兼容 Bash 的同时 （默认不兼容，除非设置成 emulate sh） 还有提供了很多改进，例如：更高效、更好的自动补全、更好的文件名展开（通配符展开）、更好的数组处理、可定制性高。


## 环境变量配置文件

macOS 默认的是 `Bourne Shell`，其环境变量配置文件及加载顺序如下：

```bash
/etc/profile
/etc/bashrc
/etc/paths
~/.bash_profile # macOS
~/.bash_login
~/.profile
~/.bashrc # linux
```

其中 `/etc/profile` `/etc/bashrc` 和 `/etc/paths` 是系统级环境变量，对所有用户都有效。但它们的加载时机有所区别：

+ `/etc/profile` 任何用户登陆时都会读取该文件
+ `/etc/bashrc` bash shell执行时，不管是何种方式，读取此文件
+ `/etc/paths` 任何用户登陆时都会读取该文件

后面几个是当前用户级的环境变量。macOS 默认用户环境变量配置文件为 `~/.bash_profile`，Linux 为 `~/.bashrc`。

如果不存在 `~/.bash_profile`，则可以自己创建一个 `~/.bash_profile`。

+ 如果 `~/.bash_profile` 文件存在，则后面的几个文件就会被忽略
+ 如果 `~/.bash_profile` 文件不存在，才会以此类推读取后面的文件


> 如果使用的是 SHELL 类型是 `zsh`，则还可能存在对应的 `/etc/zshrc` 和 `~/.zshrc`。任何用户登录 `zsh` 的时候，都会读取该文件。某个用户登录的时候，会读取其对应的 `~/.zshrc`。

## 添加环境变量

#### 系统环境变量 `/etc/paths`


一般添加系统环境变量，建议通过修改 `/etc/paths` 的方式进行添加。一般不建议直接修改 `/etc/paths` 文件，而是将路径写在 `/etc/paths.d/` 目录下的一个文件里，系统会逐一读取 `/etc/paths.d/` 下的每个文件。

`Git` 路径就是这样实现的。我们先看看 Git 的例子：

首先 `/etc/paths` 的文件内容大致如下：

```bash
$ cat /etc/paths
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin
/opt/local/bin
/opt/local/sbin
```

然后查看 `/etc/paths.d/` 目录：

```bash
$ ls -l /etc/paths.d
-rw-r--r--    1 root  wheel    19  6 10  2015 git
-rw-r--r--    1 root  wheel    17  4 20  2016 go
```

查看 `/etc/paths.d/git` 文件的内容：

```bash
$ cat /etc/paths.d/git
/usr/local/git/bin
```

`/usr/local/git/bin` 就是 Git 的可执行文件路径。

所以我们如果要添加一个系统环境变量，也可以参考这种方式。

提供一个添加环境变量的的 shell语句，其中 `/usr/local/sbin/mypath` 就是我们自己的可执行文件的路径：

```bash
$ sudo -s 'echo "/usr/local/sbin/mypath" > /etc/paths.d/mypath'
```

但添加完成之后，命令不会立即生效，有两种方法使配置文件生效：

+ 重新登录终端（如果是图形界面，即重新打开 Terminal）
+ 通过 `source` 命令加载：`source /etc/paths`

配置生效之后，就可以使用 `mypath` 命令了。

#### 系统环境变量 `/etc/profile` 和 `/etc/bashrc`

> 注：一般不建议修改这两个文件

添加环境变量的语法为：

```bash
export PATH="$PATH:<PATH 1>:<PATH 2>:<PATH 3>:...:<PATH N>"
```

所以在 `/etc/profile` 和 `/etc/bashrc` 中添加环境变量，只需要在文件中加入如下代码：

```bash
export PATH="/Users/jh/anaconda/bin:$PATH"
```

#### 用户环境变量

添加用户环境变量，只需要修改 `~/.bash_profile`（Bourne Shell）或 `~/.zshrc`（zsh）或其他用户级配置文件即可。添加环境变量的语法也是：

```bash
export PATH="$PATH:<PATH 1>:<PATH 2>:<PATH 3>:...:<PATH N>"
```

下面是我的 `~/.zshrc` 的部分配置：

```bash
export ANDROID_HOME=~/Library/Android/sdk
export PATH=$PATH:~/Library/Android/sdk/tools:~/Library/Android/sdk/platform-tools
export PATH=$PATH:/usr/local/mysql/bin:/usr/local/mysql/support-files
alias tree="find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'"
alias t="trans"
export PATH="/usr/local/sbin:$PATH"
```

可以通过 `echo $PATH` 命令查看当前环境变量：

```bash
echo $PATH
/usr/local/sbin:/Users/jh/.nvm/versions/node/v7.6.0/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/git/bin:/Users/jh/Library/Android/sdk/tools:/Users/jh/Library/Android/sdk/platform-tools:/usr/local/mysql/bin:/usr/local/mysql/support-files:/Applications/Sublime Text.app/Contents/SharedSupport/bin
```

修改了配置文件之后，依旧需要重新登录 SHELL 或者使用 `source ~/.zshrc` 来是配置立即生效。


#### export

还有一种添加环境变量的方法： `export` 命令。

`export` 命令用于设置或显示环境变量。通过 `export` 添加的环境变量仅在此次登陆周期内有效。

比如很多时候我们的开发环境和生产环境，就可以通过设置一个临时环境变量来，然后在程序中根据不同的环境变量来设置不同的参数。

```bash
# 设置 NODE_ENV 环境变量。退出 SHELL 时失效
$ export NODE_ENV=development
# 查看当前所有环境变量
$ export -p
...
typeset -x NODE_ENV=development
typeset -x USER=jh
...
```

在 Node.js 代码中判断当前环境是开发环境还是生产环境：

```javascript
if(process.env.NODE_ENV === 'development') {
    console.log('开发环境');
} else {
    console.log('生产环境');
}
```

---

Github Issues [https://github.com/nodejh/nodejh.github.io/issues/34](https://github.com/nodejh/nodejh.github.io/issues/34)
