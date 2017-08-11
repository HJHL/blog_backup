---
title: Nginx日志分析 （一）
date: 2017-08-01 13:49:10
tags:
- 安全
- 运维
---

​        由于项目需求，需要收集Linux下的各种日志（包括系统日志和应用程序日志）并对日志进行一个简单的分析。先挑了一个最熟悉的入手----Nginx日志。
<!--more-->
​        一般，Linux的各种日志会存放在`/var/log/`目录下。我的VPS使用的是[`Debian`](https://www.debian.org)，使用包管理器`apt`安装的Nginx，所产生的日志存放在`/var/log/nginx/`下。其中，`access.log`是访问日志，记录什么人在什么时间访问了什么资源，还有访问者的http状态码和user-agent。`error.log`是错误日志，记录了程序在运行过程中产生的一些错误，一般是配置不当产生的（至少我是这样，有一次写错网站的root目录，导致访问不了，浏览器直接返回一个空白页面，折腾了许久，其实只要看一下error.log文件就知道错在哪里了）

​        在查看日志的时候发现这么一段有趣的地方：

```
60.169.73.247 - - [30/Jul/2017:11:25:21 +0800] "GET http://proxy.httpdaili.com/ip.asp?ip=119.28.134.144&dk=80 HTTP/1.1" 404 168 "-" "-"
61.160.207.19 - - [30/Jul/2017:20:18:13 +0800] "GET http://180.97.33.108/?wd=119.28.134.144&dk=80 HTTP/1.1" 404 168 "-" "-"
222.186.58.57 - - [31/Jul/2017:02:37:13 +0800] "GET http://proxy.httpdaili.com/ip.asp?ip=119.28.134.144&dk=80 HTTP/1.1" 404 168 "-" "-"
1.202.68.249 - - [31/Jul/2017:12:00:48 +0800] "GET http://www.rfa.org/english/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/4
5.0.2454.101 Safari/537.36"
1.202.71.62 - - [31/Jul/2017:12:00:48 +0800] "GET http://dongtaiwang.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2
454.101 Safari/537.36"
1.202.71.62 - - [31/Jul/2017:12:00:48 +0800] "CONNECT www.baidu.com:443 HTTP/1.1" 400 172 "-" "-"
124.127.221.68 - - [31/Jul/2017:12:00:48 +0800] "GET http://www.123cha.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0
.2454.101 Safari/537.36"
106.120.40.94 - - [31/Jul/2017:12:00:49 +0800] "CONNECT boxun.com:443 HTTP/1.1" 400 172 "-" "-"
106.120.40.94 - - [31/Jul/2017:12:00:49 +0800] "CONNECT cn.bing.com:443 HTTP/1.1" 400 172 "-" "-"
124.127.223.134 - - [31/Jul/2017:12:00:49 +0800] "GET http://ip-api.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.24
54.101 Safari/537.36"
1.202.81.66 - - [31/Jul/2017:12:00:50 +0800] "CONNECT www.voanews.com:443 HTTP/1.1" 400 172 "-" "-"
106.120.33.71 - - [31/Jul/2017:12:00:51 +0800] "GET http://www.minghui.org/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0
.2454.101 Safari/537.36"
106.120.61.128 - - [31/Jul/2017:12:00:51 +0800] "GET http://www.epochtimes.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/
45.0.2454.101 Safari/537.36"
123.160.234.233 - - [31/Jul/2017:12:05:01 +0800] "GET http://dongtaiwang.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45
.0.2454.101 Safari/537.36"
121.57.228.219 - - [31/Jul/2017:12:05:01 +0800] "CONNECT boxun.com:443 HTTP/1.1" 400 172 "-" "-"
106.120.43.73 - - [31/Jul/2017:12:05:01 +0800] "GET http://www.123cha.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.
2454.101 Safari/537.36"
171.116.144.217 - - [31/Jul/2017:12:05:01 +0800] "GET http://ip-api.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.24
54.101 Safari/537.36"
1.83.125.42 - - [31/Jul/2017:12:05:01 +0800] "CONNECT cn.bing.com:443 HTTP/1.1" 400 172 "-" "-"
171.34.218.201 - - [31/Jul/2017:12:05:01 +0800] "GET http://www.epochtimes.com/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/
45.0.2454.101 Safari/537.36"
124.236.173.242 - - [31/Jul/2017:12:05:01 +0800] "GET http://www.minghui.org/ HTTP/1.1" 400 672 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45
.0.2454.101 Safari/537.36"
171.117.9.177 - - [31/Jul/2017:12:05:01 +0800] "CONNECT www.baidu.com:443 HTTP/1.1" 400 172 "-" "-"
101.249.87.159 - - [31/Jul/2017:12:05:01 +0800] "CONNECT www.voanews.com:443 HTTP/1.1" 400 172 "-" "-"
80.82.78.38 - - [31/Jul/2017:13:56:08 +0800] "GET http://www.baidu.com/cache/global/img/gs.gif HTTP/1.1" 404 168 "-" "Mozilla"
119.28.134.144 - - [31/Jul/2017:16:21:00 +0800] "GET http://www.yahoo.com HTTP/1.1" 404 168 "-" "-"
```

看到这个的一瞬间，有点萌比。一般`GET`后面接的都是`/`开始的，也就是说明资源在服务器上，从服务器上定位，但这里怎么多出来一个http协议的外链呢？！由于看这个网址`http://proxy.httpdaili.com`，猜想它可能是一个代理提供网站。尝试直接访问，出现这样的结果，![直接访问](https://coding.net/u/huizaizai/p/Images/git/raw/master/2017/080101.png)，按照一般套路，改成`www.httpdaili.com`，试了一下，果真可以，如下图：

![www.httpdaili.com](https://coding.net/u/huizaizai/p/Images/git/raw/master/2017/080102.png)

看来情况是这样：这个代理网站不停的运行扫描器，扫描网络中开放了80端口的主机，然后添加进它的代理池，导致后面的许多IP想用这个代理访问。显而易见，这个是行不通的，因为Nginx默认并没有开启代理功能，而且我也没有配置，自然就失败返回404了。从这一点来看，这个代理网站很坑，扫到80端口后直接添加进了它的代理池，没有进行有效性验证，让它的客户白忙活了。

​        这种现象是为什么呢？为什么会扫描代理的会专门扫描80端口？抱着疑惑，我尝试[`Google`](https://www.google.com)搜索了如下关键字：`Proxy Nginx`。搜到这么一篇文章：[Why do I see requests for foreign sites appearing in my log files?](https://wiki.apache.org/httpd/ProxyAbuse)。具体说了什么自己看吧，虽然说的是Apache的，不过原理性的东西，懂了就都会懂，只是实现起来一些细节的差别了。

​        最后，附上这次日志分析用上的文件：[点击下载`access.log`](https://coding.net/u/huizaizai/p/Files/git/blob/master/access.log)
