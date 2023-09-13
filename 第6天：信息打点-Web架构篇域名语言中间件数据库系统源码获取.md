# 第6天：信息打点-Web架构篇&amp;域名&amp;语言&amp;中间件&amp;数据库&amp;系统&amp;源码获取

![1645075264612-6ace8651-ad11-4228-bb76-18035a624629.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132222917-194814977.png)

```plain
#知识点：
1、打点-Web架构-语言&中间件&数据库&系统等
2、打点-Web源码-CMS开源&闭源售卖&自主研发等
开源：可以上网搜索，免费使用，使用人不太重视
闭源：在内部，或者需要购买（源代码可能加密等）
自主研发：这套源码只是自己研发，只有自己用，不想让别人知道。（大型网站）
3、打点-Web源码获取-泄露安全&资源监控&其他等
4、打点-Web域名-子域名&相似域名&反查域名&旁注等

#信息点：收集的信息
基础信息，系统信息，应用信息，防护信息，人员信息，其他信息（子公司等）等

#技术点：
CMS识别，端口扫描，CDN绕过，源码获取，子域名查询，WAF识别，负载均衡识别等
```

- #### 信息打点-个人博客-XIAODI8-架构&源码

```plain
数据包：抓包分析数据。
```

![1645075385695-ba16bc51-73d8-436a-9b7d-74cb62d50e5b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132249622-1393280984.png)

```plain
Connection: Upgrade, Keep-Alive
Content-Encoding: gzip
Content-Length: 1924
Content-Type: application/x-javascript; charset=utf-8
Date: Fri, 10 Dec 2021 12:29:22 GMT
Etag: W/d7d34ffba25c52d857e88f3845231c46
Keep-Alive: timeout=5, max=100
Product: Z-BlogPHP 1.6.6 Valyria
Server: Apache/2.4.46 (Win32) OpenSSL/1.1.1g mod_fcgid/2.3.9a
Set-Cookie: ZDEDebuggerPresent=php,phtml,php3; path=/
Upgrade: h2,h2c
Vary: Accept-Encoding
X-Powered-By: PHP/5.4.45


xiaodi8.com 
审查元素 
Apache      中间件
 php        脚本语言
 zbolg      源码名称
 Windows    操作系统（大小写）
 mysql    通过搭建组合来判断
大小写（不要在参数加，在文件中测试）在文件名判断操作系统也可以通过ping命令
Ping命令测试：根据ttl值来判定
```

![1645075425467-543ded73-7c71-4714-ae2a-62296d613f50.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132318258-554736704.png)

```plain
https://www.cnblogs.com/ziyeqingshang/p/3769542.html
数据库access mysql mssql redis mongdb oracle postgresql
数据库常见开放端口https://blog.csdn.net/SeniorShen/article/details/108643979
接口：（被动扫描）网上平台 coolaf.com/tool/port
端口扫描：
```

![1645075458931-8a19f38e-daef-412f-a9dd-f06b0d9cf316.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132332067-690868848.png)

```plain
源码:直接利用公开的漏洞进行尝试安全测试 白盒代码审计
mysql：漏洞或者弱口令等
PHP：常规的漏洞，如log4j漏洞
Apache：中间价安全等。
```

- #### 信息打点-某违法APP-面具约会-架构&源码

```plain
打开进行测试抓包（注册等）
发现host：love.youdingb.com
```

![1645075516900-0c8d3c6f-36d4-497c-bc54-5544ba3e915c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132442316-1004775200.png)

```plain
返回包：发现Nginx，PHP，Linux，mysql
```

![1645075540551-c321dde2-519e-4956-87f2-00a56b153099.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132455844-1790205180.png)

```plain
复制地址访问：
```

![1645075577589-fa6138d9-063f-4ada-848d-fd7c8d30f853.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132510518-1961832535.png)

```plain
无法注册，没办法进行下一步测试。
mysql检测：
```

![1645075600871-0d8d31ad-133d-43e3-a2b3-70f34ba990bb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132524884-468032874.png)

```plain
黑源码：关键字（棋牌源码）bing Google fofa上面搜索
https://www.huzhan.com/
https://28xin.com/
https://bbs.bcb5.com/
https://www.shixinwl.com/
https://www.lengcat.com/
https://www.xlymz.com/
https://www.ymadx.com/

利用fofa.so搜索：地址：https://www.i.youdingb.com/index.php/index/store/detail/id/74.html
```

![1645075653702-62bd3903-4477-4368-a384-1da67c0700c9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132549807-2090077323.png)

- #### 信息打点-某违法应用-爱心工程-架构&域名

```plain
http://jmlsd.com/
http://www.axgc168.com/
http://jmlsd.cn/
变动com 变成cn org cn.com 等
变动jmlsd 变成jmlsd123 jmlsd8等
找whois信息，备案信息，域名相似。后缀和名字
关键字：键镁乐时代
```

![1645075690467-41d439c9-99d0-45b0-8acb-7d160a2987ff.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132611656-835771832.png)

```plain
可以利用到网上平台查询:
```

![1645075719242-6f5fb1f8-0f6b-413b-968d-8b32ffbaa53e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132632954-1051698856.png)

```plain
https://dxy.butian.net/ 
端口扫描，数据包返回 jsp3 
大型一般都是Linux
源码：圈内人或者其他
wangyuan.com
搜索为PHP信息
```

![1645075756319-929693d9-4edb-4c8a-8213-d1a95e695e6c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910132653856-1829533496.png)

```plain
• 发现server 不懂百度。
• 这种情况下一般都会有cdn，需要找到真实IP。
全球ping,没有cdn
```