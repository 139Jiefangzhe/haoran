![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094701891-1867886297.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094709958-610329227.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094715319-536784015.png)

```plain
#知识点：
0、WAF介绍&模式&识别&防护等
1、信息收集-被动扫描&第三方接口
2、信息收集-基于爬虫&文件目录探针
3、信息收集-基于用户&代理池&白名单

#章节点：
WAF绕过主要集中在信息收集，漏洞发现，漏洞利用，权限控制四个阶段。

#补充点：
1、什么是WAF？
Web Application Firewall（web应用防火墙），一种公认的说法是“web应用防火墙通过执行一系列针对HTTP/HTTPS的安全策略来专门为web应用提供保护的一款产品。
基本可以分为以下4种
软件型WAF
以软件的形式安装在服务器上面，可以接触到服务器上的文件，因此就可以检测服务器上是否有webshell，是否有文件被创建等。

硬件型WAF
以硬件形式部署在链路中，支持多种部署方式。当串联到链路上时可以拦截恶意流量，在旁路监听模式时只记录攻击但是不进行拦截。

云 WAF
一般以反向代理的形式工作，通过配置后，使对网站的请求数据优先经过WAF主机，在WAF主机对数据进行过滤后再传给服务器

网站内置的WAF
就是来自网站内部的过滤，直接出现在网站代码中，比如说对输入的参数强制类转换啊，对输入的参数进行敏感词检测啊什么的

2、如何判断WAF？
Wafw00f，看图识别，其他项目脚本平台
https://mp.weixin.qq.com/s/3uUZKryCufQ_HcuMc8ZgQQ

3、常见WAF拓扑&防护？
见上图流量走向&常见漏洞

4、目前有哪些常见WAF产品？
参考：https://blog.csdn.net/w2sft/article/details/104533082/
① 硬件型
硬件型WAF以一个独立的硬件设备的形态存在，支持以多种方式（如透明桥接模式、旁路模式、反向代理等）部署到网络中为后端的Web应用提供安全防护，是最为传统的WAF型态，在受访企业中部署占比为35.2%。相对于软件产品类的WAF，这类产品的优点是性能好、功能全面、支持多种模式部署等，但它的价格通常比较贵。国内的绿盟、安恒、启明星辰等老牌厂商旗下的WAF都属于此类。

② 软件型
这种类型的WAF采用纯软件的方式实现，特点是安装简单，容易使用，成本低。但它的缺点也是显而易见的，除了性能受到限制外，还可能会存在兼容性、安全等问题。这类WAF的代表有ModSecurity、Naxsi、ShareWAF、安全狗等。

③ 云WAF
随着云计算技术的快速发展，使得基于云的WAF实现成为可能，在本次调查中占比甚至超过了传统的硬件WAF跃升为第一位，达到39.4%。阿里云、腾讯云、深信服云WAF、Imperva WAF是这类WAF的典型代表。

有时候碰到一些特殊情况，防护waf不同，环境不同等等这些情况，所以waf不一定能绕过。绕不过的情况的话就没有办法。
```

- 信息收集-被动扫描-黑暗引擎&三方接口

```plain
安全狗 d盾等产品是比较少的了，支持的东西比较少。

扫描分类：
1.主动扫描
2.被动扫描
有的waf会对入口，出口进行检测 出口（部分waf有）

直接去扫描网站，属于主动扫描，和waf有一个亲密的接触
通过fofa搜网站信息，属于被动扫描，和waf没有一个亲密的接触

被动收集信息是最简单的绕过waf，在菱角社区中就会有很多在线的平台去查询网站，这些都属于被动扫描。


黑暗引擎：Fofa Quake Shodan zoomeye 0.zone等
其他接口：https://forum.ywhack.com/bountytips.php?getinfo

用这种接口去查询，不会对自身访问网站进行影响。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094735004-1366951484.png)

- 信息收集-目录扫描-Python代理加载脚本

```plain
#信息收集常见检测：
1、脚本或工具速度流量快
2、脚本或工具的指纹被识别
3、脚本或工具的检测Payload


#信息收集常见方法：
1、延迟：解决请求过快封IP的情况
2、代理池：在确保速度的情况下解决请求过快封IP的拦截
3、白名单：模拟白名单模拟WAF授权测试，解决速度及测试拦截
4、模拟用户：模拟真实用户数据包请求探针，解决WAF指纹识别


目录扫描，为什么被拦截？有哪些原因
速度，UA，流量特征等

速度解决：1.延迟  2.代理池


可以通过延迟扫描来进行绕过。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094734514-100067300.png)

```plain
在这个时候扫描，网站没有封ip，也能正常打开。

也可以用代理池来进行绕过，在https://www.kuaidaili.com/中购买快代理来进行测试。

在7kbsan中没有支持代理扫描，需要用到Proxifier这个工具来进行代理转发。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094734693-1126149785.png)

```plain
但是在这里扫描还是不行，可能是由于工具的指纹头被waf记录或者其他的原因，所以需要写脚本来进行爬行。
import requests
import time

headers={
    'Connection': 'keep-alive',
    'Cache-Control': 'max-age=0',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36',
    'Sec-Fetch-Dest': 'document',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'Sec-Fetch-Site': 'none',
    'Sec-Fetch-Mode': 'navigate',
    'Sec-Fetch-User': '?1',
    'Accept-Encoding': 'gzip, deflate, br',
    'Accept-Language': 'zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7',
   'Cookie': 'bdshare_firstime=1581597934650; PHPSESSID=ncsajdvh39qse0qlsgqokshuc4; yx_auth=dc4fq8FAEkyiAUZ54b5zl9GGStCxXoRb1TFaAaozygMiSc5uZYHjR3gCQm%2BtKNz3bcjbTi8BRgcd%2F7LvR0lHN1j319CI6x29Z2QDI38',
}

for paths in open('php_b.txt',encoding='utf-8'):
    url='http://www.testxiaodi.fun/'
    paths=paths.replace('\n','')
    urls=url+paths
    proxy = {
        'http': 'tps686.kdlapi.com:15818',
    }
    try:
        code=requests.get(urls,headers=headers,proxies=proxy).status_code
        #req=requests.get(urls, headers=headers, proxies=proxy)
        #print(urls)
        #print(req.text)
        #time.sleep()
        print(urls+'|'+str(code))
        if code==200 or code==403:
            print(urls+'|'+str(code))
    except Exception as err:
        print('connecting error')
        time.sleep(3)
        
执行：G:\python38\python.exe scan_aliyunbt.py
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094734798-924118648.png)

```plain
在安全狗中，有这种UA白名单设置，如果白名单符合，那就不会进行拦截，可以模拟百度 谷歌等爬虫网站来进行测试。
```

- 信息收集-工具扫描-Awvs&Xray&Goby内置

```plain
Awvs-设置速度&加入代理
Xray-配置修改&进程转发
Goby-配置加入Socket代理


Awvs-设置速度&加入代理
再添加目录的时候，可以设置代理或者可以设置扫描速度。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094734712-283961245.png)

```plain
Xray-配置修改&进程转发

可以通过Proxifier进行代理或者在配置文件（config.yaml）上进行修改
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094735497-1716979972.png)

```plain
Goby-配置加入Socket代理
在goby中设置，然后可以设置socket代理（扫描端口，操作系统等），也可以用http来进行扫描。
goby扫描端口用的socket协议，对waf没有防护，网站不会受到影响。不是web协议，没有经过waf防火墙。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094734914-458256932.png)