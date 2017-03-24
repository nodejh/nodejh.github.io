+++
categories = ["Shadowsocks"]
date = "2017-02-10T16:52:15+08:00"
description = "在 VPS 上安装 Shadowsocks Server"
draft = false
tags = ["Shadowsocks"]
title = "Install Shadowsocks Server in VPS"

+++

首先关于 Shadowsocks 的使用说明在这里：[Shadowsocks 使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-使用说明)。

使用说明中描述的也非常详细。我主要是记录 在 Vultr 的 VPS 上安装 shadowsocks 安装使用过程中遇到的错误，以及错误解决办法。

我的 VPS 系统版本是 Ubuntu 16.10：

```
$ cat /etc/issue
Ubuntu 16.10 \n \l

# 或
$ sudo lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.10
Release:	16.10
Codename:	yakkety
```



首先安装 pip：

```
$ sudo apt-get install python-pip
```

然后通过 pip 安装 shadowsocks，结果就报错了：

```
$ sudo pip install shadowsocks
Could not import setuptools which is required to install from a source distribution.
Please install setuptools.
```

这是因为没有安装 setuptools，所以安装一下：

```
$ sudo pip install -U setuptools
# 然后再安装 shadowsocks
$ sudo pip install shadowsocks
```

更具体的内容可在 Github Issues 查看：[ImportError: No module named setuptools (add details to docs) ](https://github.com/pypa/pip/issues/1064)。

接下来配置 shadowsocks：

```
$ sudo vim /etc/shadowsocks.json
```

写入下面的配置：

```
{
    "server":"my_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"chacha20",
    "fast_open": false
}
```

Shadowsocks 的配置文件描述在 [Configuration via Config File](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)。

我的加密算法设置的是 `chacha20`。之前主流的SS有两种加密算法：`RC4-MD5` 和 `aes-256-cfb`。

`aes-256-cfb` 是各种一键包默认的加密方法，但是由于路由器和手机性能的问题，这种算法还是多少会影响到一些速度的。`RC4-MD5` 主要是加密太简单了，在 GFW 面前加密不加密已经没有什么区别...所以我们需要密码强度比 `RC4-MD5` 高,但是速度比 `aes-256-cfb` 快的加密算法,那就是 `chacha20` 了。可以说是目前性价比比较高的加密算法。

然后后台启动 shadowsocks：

```
$ sudo ssserver -c /etc/shadowsocks.json -d start
```

又报错了：

```
INFO: loading config from /etc/shadowsocks.json
Traceback (most recent call last):
  File "/usr/local/bin/ssserver", line 11, in <module>
    load_entry_point('shadowsocks==2.8.2', 'console_scripts', 'ssserver')()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/server.py", line 34, in main
    config = shell.get_config(False)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 262, in get_config
    check_config(config, is_local)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 124, in check_config
    encrypt.try_cipher(config['password'], config['method'])
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher
    Encryptor(key, method)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 83, in __init__
    random_string(self._method_info[1]))
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher
    return m[2](method, key, iv, op)
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/sodium.py", line 62, in __init__
    load_libsodium()
  File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/sodium.py", line 42, in load_libsodium
    raise Exception('libsodium not found')
Exception: libsodium not found
```

因为没有 `libsodium`，`libsodium` 是 `chacha20` 加密算法所需要的一个包。所以接下来就安装它：

```
$ wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar zxf LATEST.tar.gz
cd libsodium*
./configure
sudo make && sudo make install
```

编译的时候又报错了：

```
configure: error: no acceptable C compiler found in $PATH
```

这是因为没有 C 编译器。所以继续安装：

```
$ sudo apt-get install build-essential
# 安装成功之后再编译
$ sudo make && sudo make install
```


然后将下面的代码加入到 `/etc/ld.so.conf`：

```
include ld.so.conf.d/*.conf"
/lib
/usr/lib64
/usr/local/lib
```

再重新载入配置：

```
$ sudo ldconfig
```

接下来再启动 shadowsocks：

```
$ sudo ssserver -c /etc/shadowsocks.json -d start
```

查看 shadowsocks 日志：

```
$ tail -f /var/log/shadowsocks.log
```

然后只需要配置好客户端，就可以愉快地番茄了。

最后分享一下我的 vultr 邀请链接，通过该链接注册您将获得 `$20` ：[vultr](http://www.vultr.com/?ref=7104654-3B)。



