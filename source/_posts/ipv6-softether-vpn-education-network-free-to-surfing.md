---
title: 【详细】IPv6+SoftetherVPN搭建校园网全局免流环境
date: 2017-08-05 16:02
tags:
- 科学上网
- 教程
- Sofether VPN
- IPv6
---
> vultr有20美元的优惠链接在下面哦！

>2016年11月26日更新：已替换最新版服务端程序链接

> 2017年8月5日更新：修改最新服务端程序链接、删除部分失效图片。

<!--more-->
# 为什么会有这篇文章
在大部分高校，IPv4一般是计流量收费的，而且，由于校园的特殊性，相应的价格也比市面上宽带服务商要高。以笔者学校为例，一个月免费3G，有两种套餐：30元40G、40元50G，超出之后想要上网的话只能以3块钱一个G的价格购买流量，对于那些喜欢看视频、下载的同学来说，每个月得承受一笔不小的流量费用，有的学校更加，每个月最多能够使用那么些流量，超出之后哪怕花钱也不能买了，然后只得用手机卡开热点。万幸的是，这些高校一般具有IPv6网络环境，并且由于国家的大力支持，普及范围广，而且不计算流量，聪明的人早已经想到通过建立在IPv6上的PT来分享资源，以缓解流量不足带来的尴尬。不过，那必须得是想要下载的资源在PT站里面有才可以，要是一些资源在上面没有，而有迫切需要的话，就得另找法子了。所以，有没有一种方法能够让我们不花费巨额的IPv4流量费用而畅想互联网呢？当然有！鉴于目前公网的环境普遍是IPv4，我们可以找一台同时具有IPv4和IPv6地址的服务器，我们在校内通过IPv6访问服务器，然后服务器处理我们的访问请求以IPv4/IPv6双栈的方式代替我们访问互联网，再将数据通过IPv6反馈给我们，从而到达免流上网的目的，并且，考虑到大部分高校IPv6没有限制速度，理论上可以达到12.5M/S甚至是125M/S的网速，当然，具体取决于你们学校的IPv6出口带宽和服务器的带宽，等等，这里就不详细列举了。

# 为什么会用softether VPN
其实有很多支持IPv6的代理工具，比如shadowsocks、OpenVPN，可是shadowsocks不能全局代理，只能代理支持Socks5代理的应用，设置起来也麻烦，OpenVPN服务端配置起来略显复杂，小白可能操作起来困难些，而SoftEther VPN是日本筑波大学的一个研究项目，它包括服务器端、客户端、服务器端管理工具等数个软件，支持 SSL-VPN (SoftEther VPN) 协议、 L2TP/IPsec 协议、 OpenVPN 协议和 Microsoft SSTP 协议。安装简单，可以全局代理！十分方便！

# 适合对象
具有IPv6地址、IPv4流量收费贵IPv6不计费的高校的爱折腾的大学生。不推荐打国服游戏的玩家使用这个，不推荐对延迟要求较高的同学使用这个。

# 使用条件、材料
- **具有IPv6网络环境**
- **一台电脑**
- **一台同时具有IPv4/IPv6地址的服务器**
- **灵巧的双手**
- **爱折腾的心**

# 搭建开始

### 服务器的选购
首先，我们得有一台**同时具有IPv4和IPv6地址的服务器**，俗称VPS(Visual Personal Server 虚拟专用服务器），一般该怎么选择呢，目前，国内的IDC提供商极少见（应该可以说没有）能够提供IPv6地址的，所以就不用考虑国内的了，买之前，最好是能够找到一个测试的IPv6地址，ping一下看看延迟高低，再根据自己的情况选择合适的VPS，一般考虑的主要是服务器的带宽、延迟、每个月可供使用的流量，当然，更重要的一点是看看这家IDC的口碑，毕竟你也不想买了没几天他就跑路吧！我在这里给大家推荐几款我自己测试过了的IDC给大家参考（当然，我无耻的放上了我的邀~不喜勿喷哦~~）：

#### [搬瓦工][1]
![搬瓦工][2]
搬瓦工这个名字听上去是不是让你感觉一阵乡土气息铺面而来呢？你是不是认为这是国人办的呢？可能你猜错了，搬瓦工其实是由一群技术牛逼的歪果仁创办的，这个中文名其实是音译过来的呢，至于为什么说是‘牛逼’，那是因为他们的VPS是基于OpenVZ虚拟化的，这种虚拟化技术容易给官方留下超售的机会（想要详细请Google），大部分使用OpenVZ架构的服务商都或多或少会超售，这时候就要看各家的技术了，搬瓦工官方承认自己超售严重，但是他们有自己独特的技术，能够保证服务器的稳定，所以超售一点也没关系啦。其实这是官方的说法，不过根据我的体验来看，他们做的确实很不错，用了这么久我也就有一次因为CPU占用率过高而导致服务器被限制了一会（其实没有啥太大的影响），并没有封我的服务器，这比其他的好多了，其他的只要你稍微超的时间久一点，然后你的机器就会suspended，更严重的直接封掉，上次我就是在其他的提供商那里的一台128M内存的小鸡上编译一个程序，导致VPS被封，和客服交流了三天，最后客服和我道歉是他们弄错了，因为本来CPU超用之后及时联系客服VPS马上就可以解封的，可是他们却误以为我们有付款，导致我和他扯了三天（主要原因是他们回复TK的速度太慢），真是倒霉。总的来说，搬瓦工是使用OpenVZ虚拟化技术的IDC中做的很不错的一家了，早期他们以便宜出名，现在便宜的套餐都取消了，最低是19.99美元一年的，性价比一般般了。不过他们在2014之后支持使用支付宝付款！更加方便了有木有！并且他们还提供30天无条件即刻退款，只要你买了之后不满意就可以直接退货！非常有保障！不过记住机会只有一次！且用且珍惜！
支付方式：支付宝、Paypal、信用卡
官网地址：[www.bandwagonhost.com][3]

#### [HostUS][4]
![HostUS][5]
HostUS最初是以提供低价、低延迟、大带宽的香港和新加坡的VPS而出名的，可是自从SL线路调整之后，对电信用户来说，现在它的香港和新加坡的VPS都已经绕路了，延迟很高，校园网用户的情况也差不多，IPv6还是绕路，具体可以去官网的测试页面看一下，官方在每个Location都提供了一个测试IPv6，不妨去测试一下看看延迟怎么样。值得一提的是，他们的美国特价套餐还是具有一定的吸引力，12美元一年，可选洛杉矶机房（就地理位置而言，洛杉矶的机房一般延迟低），2T流量每月，768M内存，性价比挺高的大家不妨考虑一下。
支付方式：Paypal、信用卡
官网地址：[www.hostus.us][3]

#### [DigitalOcean][6]
不限流量！不限流量！不限流量！重要的事情说三遍！尽管官方套餐上标明一个月1T的流量，但是你会发现后台没有统计流量的菜单，发TK问客服也确认了此事（至少目前是这样）。这家的机器是KVM虚拟化的（不懂虚拟化技术的可以Google一下他们的区别，一般来说OpenVZ劣于KVM，KVM的VPS一般也会贵点），所以可以使用锐速加速！（关于这个，之后我会再写一篇使用锐速加快网速的教程），最低五美元一个月，按时按量计费！相当灵活！而且在Github education上有一个Student Package，可以领到50美元的优惠码，点击这个[链接][6]进去还可以额外获得十美元的奖励哦！（当然，你购买VPS之后我也会得到一点实惠，利人利己！^_^）不过，必须得先充值五美元才能够进行VPS的购买！这点要注意！（其实想想已经很划算了，人民币三十左右就可以玩一年，多爽！）额，但是有一点值得注意，[DigitalOcean][6]（简称DO）支持信用卡和paypal支付，但是只支持绑定了信用卡的paypal账户支付！所以还是逃脱不了信用卡，这点比较恼人！（不过这样确实对避免了国人把它玩坏有一点帮助）下面要说的[Vultr][7]也是这样，只支持信用卡支付。（如果实在想购买可以联系我代购）
支付方式：信用卡、paypal
官网地址：[www.digitalocean.com][6]

#### [Vultr][7]
[Vultr][7]其实也没啥可介绍的，它的运营模式和[DO][6]差不太多，只是机房的位置和[DO][6]有所不同，[DO][6]在新加坡有机房，[Vultr][7]在日本有机房，不管都被玩坏了，电信联通绕路，基本上没啥好考虑的。还是老老实实选择美国的机房吧，推荐洛杉矶！不过[Vultr][7]有一个信用卡用户可以参与的[活动][8]（[点击][8]进入），通过这个链接注册可以获得五十美元，但是必须在两个月之内用完，稍微有点坑。还有一个20美元的[优惠码][7]（[NGINX20][7]），一年的有效期，仅限新用户注册时使用，没有信用卡有paypal也可以获得20美元。[Vultr][7]的一大特色就是可以自定义ISO，所以你可以自己尝试安装windows系统！
支付方式：paypal、信用卡
官网地址：[www.vultr.com][7]
**[【点击这里】](http://www.vultr.com/?ref=6949620-3B)**可以获得**20**美元到账！

### 操作步骤
买好VPS之后，就可以开始搭建了，有几点要注意：搬瓦工购买之后后台可以添加VPS，[Vultr][7]也是一样，[DO][6]购买时最好选择【enable IPv6】，避免麻烦！**注：我所推荐的这几个都是经过我测试后，确定有IPv6地址的，如果想去购买其他的VPS，事先要明确是否提供IPv6地址！**
操作系统我选择的是Debian 7 64位，你选则CentOS、Ubuntu也行，但最好和我一样，避免麻烦，教程所使用的是[HostUS][4]香港的VPS（虽然已经废了，但是用来写个教程还是很不错滴）

1、安装好系统后，ssh到VPS（这个网上一大堆，不懂的Google），首先更新一下软件源
> apt-get update

2、安装softether VPN所依赖的软件和库
不清楚缺少什么我们可以通过执行安装脚本来自动检测，再手动添加上就行了！用wget命令下载软件包，然后再解压，切换进入解压后的文件夹，执行安装脚本，即是下面的代码（按照顺序来！！）

安装编译环境（**在VPS上操作**）
> apt-get install build-essential

下载服务端程序
> wget http://www.softether-download.com/files/softether/v4.22-9634-beta-2016.11.27-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.22-9634-beta-2016.11.27-linux-x64-64bit.tar.gz

解压
> tar -zxvf softether-vpnserver-v4.22-9634-beta-2016.11.27-linux-x64-64bit.tar.gz

进入解压后的目录
> cd vpnserver/

执行安装脚本
> ./.install.sh

然后他会向你提出三个问题，都输入“1”再回车。



出现报错就阅读报错信息解决问题，直到提示安装成功（一大段英文，耐心点看）。
然后就安装完成啦！不过还没完，还需要设置一下

来来来，先设置一下语言
> vim lang.config

将最后一行改成cn，然后保存退出

启动服务
> ./vpnserver start

然后用管理软件登陆，点击【新设置】，填入相应内容，然后点击确定，再连接。
第一次连接会出现这个
![设置管理密码](http://upload-images.jianshu.io/upload_images/1593532-d8cba5475cc9bfd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是让你设置一个管理密码，填自己想填的就好,然后点确定，登陆。
第一次回登陆完成后，会出现这个

![初始化_1](http://upload-images.jianshu.io/upload_images/1593532-7f0ff186238d45b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再弹出来这个

![初始化_2](http://upload-images.jianshu.io/upload_images/1593532-f3138b081633226a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还是点击确定

![初始化_3](http://upload-images.jianshu.io/upload_images/1593532-e6d61b6535ff6494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还是点击确定（够傻瓜化吧？）
然后比较重要的来了，

![初始化_4](http://upload-images.jianshu.io/upload_images/1593532-2ff992efb2b5cca0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![初始化_5](http://upload-images.jianshu.io/upload_images/1593532-b7527dbbd482a068.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![初始化_6](http://upload-images.jianshu.io/upload_images/1593532-d6a10c123440beb1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

弹出来这个，不要被它吓到了，点击确定吧！（一定要这样，要不然连接VPN的时候就会因为无法获取到IP地址而导致无法连接成功）

![初始化_7](http://upload-images.jianshu.io/upload_images/1593532-8a14345ed27619d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好了，现在可以新建一个用户测试一下了。如下操作：

![添加用户_1](http://upload-images.jianshu.io/upload_images/1593532-4038628f7737b8b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加用户_2](http://upload-images.jianshu.io/upload_images/1593532-307d6de29c63c1fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![添加用户_3](http://upload-images.jianshu.io/upload_images/1593532-d268c6240fc6d0a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后点击确定就算大功告成啦！
至此，服务器端就配置的差不多了。

### 连接VPN
以windows 10为例

点击【添加VPN连接】
![新建VPN](http://upload-images.jianshu.io/upload_images/1593532-bffb0fa585c7a1ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后按照下面的这个来填

![设置VPN](http://upload-images.jianshu.io/upload_images/1593532-350e99b6b3d9fa54.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意这里，别选错了。
![设置VPN-注意](http://upload-images.jianshu.io/upload_images/1593532-f67cc23a1fbf9f6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后我们连接这个VPN

![大功告成](http://upload-images.jianshu.io/upload_images/1593532-1a2b3185422f0561.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后就尽情的享受互联网吧！^_^

多说一句：由于选择的是国外的服务器，所以，也可以达到科学上网的目的。如果只是为了科学上网或者是玩外服游戏的需求的话，本文也是一个不错的选择，具体看你怎么用啦！

Echo 
二零一六年五月七日



[1]:https://bandwagonhost.com/aff.php?aff=6859
[2]:http://i3.buimg.com/0583bce10a81e272.png
[3]:https://bandwagonhost.com/aff.php?aff=6859
[4]:https://my.hostus.us/aff.php?aff=1188
[5]:http://i3.buimg.com/a7c8fb92cd06ca4a.png
[6]:https://m.do.co/c/2efe6dd011aa
[7]:http://www.vultr.com/?ref=6890151
[8]:https://www.vultr.com/freetrial/
[9]:http://i4.buimg.com/63fd0cc2bab010ef.png
[10]:http://i4.buimg.com/75835130971bc713.png
[11]:http://i4.buimg.com/9777af8c148917eb.png
[12]:http://i4.buimg.com/f2658168b3cc46cd.png
[13]:http://i4.buimg.com/cea4d6c216a38d8d.png
[14]:http://i4.buimg.com/77df8e0cb5904cd2.png
[15]:http://i4.buimg.com/881bc282fcd78e08.png
[16]:http://i4.buimg.com/3269431b1dadb9c4.png
[17]:http://i4.buimg.com/c99119a29b49eaf5.png
[18]:http://i2.buimg.com/50fe5d1a1e0bc878.png
