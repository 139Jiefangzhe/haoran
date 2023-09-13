# 第9天：信息打点-CDN绕过篇&amp;漏洞回链&amp;接口探针&amp;全网扫描&amp;反向邮件

![1645077684779-679e0e89-25fa-4246-b785-c730ac2c118c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910173127249-490545150.png)

```plain
#知识点：
0、CDN知识-工作原理及阻碍
1、CDN配置-域名&区域&类型
2、CDN绕过-靠谱十余种技战法
3、CDN绑定-HOSTS绑定指向访问
hosts路径：C:\Windows\System32\drivers\etc
#前置知识：
1.传统访问（没有CDN）：用户访问域名–>解析服务器IP–>访问目标主机
2.普通CDN：用户访问域名–>CDN节点–>真实服务器IP–>访问目标主机
3.带WAF的CDN：用户访问域名–>CDN节点（WAF）–>真实服务器IP–>访问目标主机

#什么是CDN
CDN的全称是Content Delivery Network，即内容分发网络。CDN是构建在现有网络基础之上的智能虚拟网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。
```

![1645077813393-db2402b4-ca8c-4479-a5c2-a3a85d8b6fa7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174111768-394823588.png)

```plain
#CDN配置：
配置1：加速域名-需要启用加速的域名（www.xiaodi8.com、*.xioadi8.com）
配置2：加速区域-需要启用加速的地区（中国，外国等）
配置3：加速类型-需要启用加速的资源（图片、小文件、全站）

#判定标准：
nslookup,各地ping（出现多个IP即启用CDN服务）

#参考知识：
https://zhuanlan.zhihu.com/p/33440472
https://www.cnblogs.com/blacksunny/p/5771827.html
子域名，去掉www，邮件服务器，国外访问，证书查询，APP抓包
黑暗空间引擎，通过漏洞或泄露获取，扫全网，以量打量，第三方接口查询等

#案例资源：
超级Ping：https://www.17ce.com/
接口查询：https://get-site-ip.com/
国外请求：https://tools.ipip.net/cdn.php
全网扫描：https://github.com/Tai7sy/fuckcdn
```

- #### 真实应用-CDN绕过-漏洞&遗留文件

```plain
历史记录：
去年的时候没有开通，但是今年就有开通。
基于:主动连接
遗留文件：
网站下面含有phpinfo.php文件
www.yansiqi.com/phpinfo.php
这个配置文件会泄露本身IP地址，将本地的地址给到你。php脚本主动把IP返回。

漏洞：
http://www.yansiqi.com/ssrf.php
输入地址，服务器自动会请求这个地址 http://47.94.236.117:8000 这个
47.94.236.117运行：python2 -m SimpleHTTPServer 8000
47.94.236.117开通了web服务，记录日志 www.yansiqi.com 存在ssrf漏洞（会接受用户的数据并利用服务器去请求）
漏洞请求47.94.236.117日志就会记录访问IP
ping去请求网站，你自己的请求。所以cdn节点 对方自己的服务器请求你的IP，就会显示IP
```

- #### 真实应用-CDN绕过-子域名查询操作

```plain
加速域名上面导致的问题：如果设置是www.yansiqi.com，这样只加速www这个域名 如果有test.yansiqi.com 没有加速，是真是IP。一般子域名极有可能和主站保持同一个IP

https://www.sp910.com/
www.sp910.com
假如这个加速域名设置：www.sp910.com 加速
我们访问sp910.com  没加速
ping sp910.com这样就会显示出真是IP 119.23.136.136
ping www.sp910.com 不会显示真是IP

ping www.sp910.com:
```

![1645077925400-4709a45f-1673-4c31-a85a-a4efdc91a35d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174142824-1077802311.png)

```plain
ping sp910.com：
```

- #### 真实应用-CDN绕过-接口查询国外访问

```plain
配置的区域可能只是中国大陆，在国外访问就是真是IP
国外访问：www.xuexila.com 119.132.147.27
```

![1645077989015-eb5ed1b9-a1d7-48df-b659-bd06c1f8f048.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174203129-1975985780.png)

```plain
国内访问：www.xuexila.com 180.76.199.46
经过再次查找，找不到了：
```

![1645078011059-3959373f-4b2a-4f8f-9e7d-745f587c2d38.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174216002-836140255.png)

- #### 真实应用-CDN绕过-主动邮件配合备案

```plain
很多站点都有发送邮件sendmail的功能，如Rss邮件订阅等。而且一般的邮件系统很多都是在内部，没有经过CDN的解析。可在邮件源码里面就会包含服务器的真实 IP。是一个主动连接。

mozhe.cn
怎么样网站发邮箱给你：
邮箱找回验证码，显示邮件原文 219.153.49.169
大概率，不是百分百真是IP
如果又不确定，就进行社工判定，查看归属地
```

![1645078052201-c383036f-9f1a-4e6a-845d-ae373ca4cc6a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174234059-1291907519.png)

- #### 真实应用-CDN绕过-全网扫描FuckCDN

```plain
资源加速问题：只是加速图片或者其他的，没有加速全站

运行fuckcdn.exe，设置ip.txt set.ini两个文件，运行输入访问的cdn节点IP+端口

配置set.ini文件：设置目标，findstr是设置关键字。
```

![1645078090549-e329ac9d-4eec-4747-99f3-acf170590498.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174252624-1257415397.png)

```plain
配置ip.txt：设置IP集合，https://www.ipip.net/ip.html
```

![1645078114777-8e3854b9-0551-46e7-9338-b522bd9caad2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230910174303062-156529901.png)

```plain
完成后出现result_ip.txt文件。
```