![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094334964-126416868.png)

```plain
#知识点：
1、浏览器插件&BurpSuite插件
2、Hack-Tools&pentestkit&fofa_view等
3、Fiora&Spring&Fastjson&Shiro&Log4j等
市面上有很多漏扫系统工具脚本，课程讲到的基本都是目前主流推荐的优秀项目！
具体项目：Burpsuite，Awvs，Xray，Goby，Afrog，Vulmap，Pocassist，Nessus，Nuclei，Pentestkit，Nexpose，BP插件(HaE,ShiroScan.FastJsonScan,Log4j2Scan等)。

#章节点：
1、漏洞发现-Web&框架层面
2、漏洞发现-服务&中间件层面
3、漏洞发现-APP&小程序层面
4、漏洞发现-PC操作系统层面
#浏览器插件-辅助&资产&漏洞库-Hack-Tools&Fofa_view&Pentestkit
资产：https://github.com/fofapro/fofa_view
辅助：https://github.com/LasCC/Hack-Tools
漏洞库：https://github.com/DenisPodgurskii/pentestkit

#BurpSuite插件-被动&特定扫描-Fiora&Spring&Fastjson&Shiro&Log4j
https://github.com/bit4woo/Fiora
https://github.com/metaStor/SpringScan
https://github.com/Maskhe/FastjsonScan
https://github.com/bigsizeme/Log4j-check
https://github.com/pmiaowu/BurpShiroPassiveScan
https://github.com/projectdiscovery/nuclei-burp-plugin
```

- 浏览器插件-辅助&资产&漏洞库-Hack-Tools&Fofa_view&Pentestkit

Hack-Tools

```plain
这个是一个工具，相当于简易继承了一些命令行，反弹shell等命令，相当于菱角社区的那个功能。
有Reverse shell，PHP Reverse Shell，TTY Spawn Shell，Useful Linux command for your Penetration Testing，Powershell handy commands，File Transfer，LFI，Cross Site Scripting (XSS)，SQL Injection，Data Encoding等功能。

这些功能能对我们简单的渗透测试有一定的帮助。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356810-492856354.png)

Fofa_view

```plain
这个是借助fofa的登录状态来进行扫描，对当前的网站进行简易的收集，比如端口，URL，HOST等信息进行收集。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356771-439934271.png)

Pentestkit

```plain
好像安装失败，项目地址：https://github.com/DenisPodgurskii/pentestkit

这个是对网站进行建议的曾经cve的信息收集，收集他曾经爆出过的cve等。
```

- BurpSuite插件-被动&特定扫描-Fiora&Spring&Fastjson&Shiro&Log4j

```plain
目标： 
1.用其他的工具项目去检测漏洞
按照工具项目访问特定的目标，可能会进行访问不了，或者访问出错，和正常访问不同。

2.用burpsuite抓包后代入插件检测
有部分目标需要特定的数据包访问的才能触发，在这里基础上在进行检测的话，访问是正常的。在burpsuite的数据包都是满足正常逻辑的数据包，能正常触发。
区别：
登录状态和未登录状态
不同目录访问的数据包的不同
1.APP抓不到web

先搞清楚 是app不走web还是你不会抓（证书检测，反代理等）
不走web的话，就有可能不是网站 url 域名 burp上面没有数据包，插件空白啥都没有，但是也会有ip地址，找到后可以进行端口扫描，服务探针，如果web 在访问网站，然后在到burp进行测试。	
```

Fiora

```plain
安装fiora，把这个jar文件安装到Extender中，然后就会出现多一个模块Fiora模块。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094357143-239077929.png)

```plain
  在https://vulfocus.cn/#/dashboard中找一个log4j的漏洞
  
  抓包，把数据包发送到send URLto fiora
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094357177-23257021.png)

```plain
发送过去后，在Fiora上的右下角就会有目标信息。如果需要全部漏洞测试的话，就直接运行，如果对单个漏洞测试的话，那就搜索漏洞poc。
如果对特定poc进行检测，那么选中他，右键run this poc。只用到poc进行测试
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356562-1633296099.png)

```plain
还可以运行这个标签来进行poc的验证run pocs with tags，在这里的标签有：cve,cve2021,rce,oast,log4j,injection

比较推荐标签进行检测
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356716-1667630726.png)

```plain
但是这个没有检测到，就很离谱，可能是工具的问题，或者漏洞点没有弄清楚等。会有很多问题。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356604-1332846507.png)

Spring

```plain
只要数据包经过，也会代入这个插件进行查询。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356364-490988658.png)

Fastjson

```plain
数据特点就是用json数据格式去发送。这就是最简单的判断。

启动靶场，抓取数据包，发送到send to fastjsonscan模块中。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356578-2045177948.png)

```plain
然后fastjson模块就会对其进行检测，但是这里检测不出来。这要对这个漏洞进行理解。才能触发这个漏洞。
这个项目只会检测content-type为xml、json、url-encoded的post发送数据包，如果是其他的话会提示not supported

但是这个没有发现，就离谱。。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356352-1672804734.png)

Shiro

```plain
只要数据包经过burp，这个插件就会自动检测。只要检测到有shiro这个组件，会自动进行检测。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094356675-1452214947.png)

Log4j

```plain
log4j也是一样，只要检测有这个插件，那么就会自动发送，也可以进行手动发送去检测数据包。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094400878-1854007974.png)