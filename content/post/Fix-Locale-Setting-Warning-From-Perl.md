+++
categories = ["Ubuntu"]
date = "2016-12-02T15:58:06+08:00"
description = "远程登录 VPS 语言错误"
draft = false
tags = ["Ubuntu"]
title = "Fix Locale Setting Warning From Perl"

+++

当使在 VPS 上安装软件的时候，经常遇到同一个警告，如下：

```
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = "en_US:",
	LC_ALL = (unset),
	LC_CTYPE = "zh_CN.UTF-8",
	LANG = "en_US"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US").
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
```

那是因为安装软件时，都会去执行一个 `update-locale` 的命令，用来更新 `locale`。

这个命令是一个脚本，用 perl 写的，可以用 `whereis update-locale` 查到，位置在 `/usr/sbin/update-locale`。

上述报错并不是因为 `update-locale` 命令而引起，`update-locale` 这段脚本没有问题，而是因为perl。

可以使用以下命令测试：

```
perl -e exit
```

其实，真正的原因是 perl 为系统使用 `zh_CN.UTF-8`，但系统并没有安装 `zh_CN.UTF-8`。

这种情况一般是vps比较常见，因为一般都是用 ssh 的方式连接到 vps 上的 `sshd` 有这个机制，会把客户机上的语言环境带到远程的机器上。

客户机一般都会设置 `zh_CN.UTF-8` 语言，用来显示中文，而远端的vps一般就只有en_US.UTF-8，zh_CN.UTF-8一旦带过去就会报找不到的错误。

所以安装一个中文语言，perl就不会报错了。

```
$ sudo locale-gen zh_CN zh_CN.UTF-8
Generating locales...
  zh_CN.GB2312... done
  zh_CN.UTF-8... done
Generation complete.
$ sudo dpkg-reconfigure locales
Generating locales...
  en_US.UTF-8... done
  zh_CN.GB2312... up-to-date
  zh_CN.UTF-8... up-to-date
Generation complete.
```

这个时候，还可以使用 `locale` 当前有哪些语言：

```
$ locale
```


---

参考

+ [How to fix a locale setting warning from Perl](http://stackoverflow.com/questions/2499794/how-to-fix-a-locale-setting-warning-from-perl)
+ [perl: warning: Setting locale failed.](https://ubuntuforums.org/showthread.php?t=1346581)


