---
layout: post
title: "给 Tengine 加上 lua 拓展"
date: 2015-10-29 22:45
comments: true
categories: System
description: Tengine, lua-nginx-module
---


Tengine 能动态加载第三方模块，成为我们青睐的选择，我们可以编译动态链接文件，而不需要重新安装 Nginx, 这对在线增强 webservice 很有帮助. 
感谢 agentzh, [lua-nginx-module](https://github.com/openresty/lua-nginx-module), 可以让我们使用 lua 增强nginx的功能, 不言而喻，我们必须现有 Lua 的环境，才能运行 ngx_lua;

## 编译 nginx_lua

官方推荐使用LuaJit,虽然也可以使用Lua，但是即时编译(Just-In-Time Compiler)混合了动态编译和静态编译，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能损耗。

* 下载安装

```
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
tar zxvf LuaJIT-2.0.4.tar.gz
cd LuaJIT-2.0.4
make
make install
```

* 设置环境变量

```
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0
```


* 然后编译ngx-lua-module.so

```
/usr/local/share/dso_tool/ --path=/Path/To/Lua-Nginx-module
```

* 设置动态库

```
> echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
> ldconfig
```

### 在 Tengine 中启用

`nginx.conf` 中先加载动态库

```
dso {
    load ngx_load_module;
}
```

在 nginx.conf 中添加

```
lua_code_cache on/off;
```

来开启是否将代码缓存，所以每次变更 "*.lua" 文件时，必须重启 nginx 才可生效.


### 使用 ngx_lua_waf

有了基础环境，我们要开始发挥 ngx lua 的优点了, 我用他安装了 waf (web application firework)
[ngx_lua_waf](https://github.com/loveshell/ngx_lua_waf)，这是一个通过 ngx_lua 编写的 web 应用防火墙, 在使用过程中也发现了 ngx_lua_waf 一个bug，给他提了一个[Pull Request](https://github.com/loveshell/ngx_lua_waf/pull/70), 码农生涯第一个 PR.


----

注：
静态编译的程序在执行前全部被翻译为机器码，而动态直译执行的则是一句一句边运行边翻译。



