![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140025684-800453064.png)

```plain
#知识点：
1、Burp 简单介绍&使用说明
2、Xray 简单介绍&使用说明
3、Awvs 简单介绍&使用说明
4、Goby 简单介绍&使用说明
5、Afrog 简单介绍&使用说明
6、Vulmap 简单介绍&使用说明
7、Pocassist 简单介绍&使用说明
8、掌握工具安装使用&原理&联动&适用
市面上有很多漏扫系统工具脚本，课程讲到的基本都是目前主流推荐的优秀项目！
具体项目：Burpsuite，Awvs，Xray，Goby，Afrog，Vulmap，Pocassist，Nessus，Nuclei，Pentestkit，Kunyu，BP插件(HaE,ShiroScan.FastJsonScan,Log4j2Scan等)等。

#章节点：
1、漏洞发现-Web&框架层面
2、漏洞发现-服务&中间件层面
3、漏洞发现-APP&小程序层面
4、漏洞发现-PC操作系统层面

Acunetix一款商业的Web漏洞扫描程序，它可以检查Web应用程序中的漏洞，如SQL注入、跨站脚本攻击、身份验证页上的弱口令长度等。它拥有一个操作方便的图形用户界面，并且能够创建专业级的Web站点安全审核报告。新版本集成了漏洞管理功能来扩展企业全面管理、优先级和控制漏洞威胁的能力。

Burp Suite是用于攻击web 应用程序的集成平台，包含了许多工具。Burp Suite为这些工具设计了许多接口，以加快攻击应用程序的过程。所有工具都共享一个请求，并能处理对应的HTTP 消息、持久性、认证、代理、日志、警报。

pocassist是一个 Golang 编写的全新开源漏洞测试框架。实现对poc的在线编辑、管理、测试。如果你不想撸代码，又想实现poc的逻辑，又想在线对靶机快速测试，那就使用pocassist吧。完全兼容xray，但又不仅仅是xray。除了支持定义目录级漏洞poc，还支持服务器级漏洞、参数级漏洞、url级漏洞以及对页面内容检测，如果以上还不满足你的需求，还支持加载自定义脚本。这个相当于poc库。

afrog 是一款性能卓越、快速稳定、PoC 可定制的漏洞扫描（挖洞）工具，PoC 涉及 CVE、CNVD、默认口令、信息泄露、指纹识别、未授权访问、任意文件读取、命令执行等多种漏洞类型，帮助网络安全从业者快速验证并及时修复漏洞。

Xray是从长亭洞鉴核心引擎中提取出的社区版漏洞扫描神器，支持主动、被动多种扫描方式，自备盲打平台、可以灵活定义 POC，功能丰富，调用简单，支持Windows /macOS /Linux 多种操作系统，可以满足广大安全从业者的自动化 Web 漏洞探测需求。

Goby是一款新的网络安全测试工具，由赵武Zwell（Pangolin、JSky、FOFA作者）打造，它能够针对一个目标企业梳理最全的攻击面信息，同时能进行高效、实战化漏洞扫描，并快速的从一个验证入口点，切换到横向。能通过智能自动化方式，帮助安全入门者熟悉靶场攻防，帮助攻防服务者、渗透人员更快的拿下目标。

Vulmap是一款 web 漏洞扫描和验证工具, 可对 webapps 进行漏洞扫描, 并且具备漏洞利用功能, 目前支持的 webapps 包括 activemq, flink, shiro, solr, struts2, tomcat, unomi, drupal, elasticsearch, fastjson, jenkins, nexus, weblogic, jboss, spring, thinkphp

项目资源：
https://www.ddosi.org/awvs14-6-log4j-rce/
https://github.com/chaitin/xray/releases
https://github.com/zan8in/afrog/releases
https://github.com/zhzyker/vulmap/releases
https://github.com/jweny/pocassist/releases
https://github.com/gobysec/Goby/releases

其他特扫：
1、GUI_TOOLS_V6.1_by安全圈小王子--bugfixed
2、CMS漏洞扫描器名称	支持的CMS平台
Droopescan	WordPress，Joomla，Drupal，Moodle，SilverStripe
CMSmap	WordPress，Joomla，Drupal，Moodle
CMSeeK	WordPress，Joomla，Drupal等
WPXF	WordPress
WPScan	WordPress
WPSeku	WordPress
WPForce	WordPress
JoomScan	Joomla
JoomlaVS	Joomla
JScanner	Joomla
Drupwn	Drupal
Typo3Scan	Typo3
致远OA综合利用工具	https://github.com/Summer177/seeyon_exp	seeyon_exp
通达OA综合利用工具	https://github.com/xinyu2428/TDOA_RCE	TDOA_RCE
蓝凌OA漏洞利用工具/前台无条件RCE/文件写入	https://github.com/yuanhaiGreg/LandrayExploit	LandrayExploit
泛微OA漏洞综合利用脚本	https://github.com/z1un/weaver_exp	weaver_exp
锐捷网络EG易网关RCE批量安全检测	https://github.com/Tas9er/EgGateWayGetShell	EgGateWayGetShell
CMSmap 针对流行CMS进行安全扫描的工具	https://github.com/Dionach/CMSmap	CMSmap
使用Go开发的WordPress漏洞扫描工具	https://github.com/blackbinn/wprecon	wprecon
一个 Ruby 框架，旨在帮助对 WordPress 系统进行渗透测试	https://github.com/rastating/wordpress-exploit-framework	wordpress-exploit-framework
WPScan WordPress 安全扫描器	https://github.com/wpscanteam/wpscan	wpscan
WPForce Wordpress 攻击套件	https://github.com/n00py/WPForce	WPForce
AWVS优缺点：
1.网站源码采用自己开发搭建
采用AWVS可以扫描

2.网站源码采用开源CMS搭建
一般用AWVS不能扫描到

冷门，小众，国内的CMS源码，用AWVS不适合。
常规漏洞扫描-自己开发搭建
常规漏洞扫描-开源CMS团队可以做这个事情，一般用工具直接找到这个漏洞不太现实（其他漏洞除外，非源码漏洞如中间件等）
举个例子，如果是thinkphp开发，那么用AWVS去扫描就没有用。
```

- 某APP-Web扫描-常规&联动-Burp&Awvs&Xray

AWVS

```plain
测试扫描网站：http://testphp.vulnweb.com/

修改爬虫UA
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140049471-716402222.png)

```plain
还可以用账号密码尝试登录去进行扫描

在网站扫描的时候，需要登录才能看到界面。没有登录跟登录的资产不同，发现漏洞机会更高。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140049445-246106613.png)

```plain
在advanced中，有自定义头跟自定义cookie，如果有登录验证码，那么就需要用这个来进行直接登录。headers中是app对比，识别指纹头
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140049337-624353139.png)

Xray

```plain
https://docs.xray.cool/#/

用两种模式：
1.主动扫描
2.被动扫描


使用语法：
USAGE:
    [global options] command [command options] [arguments...]

COMMANDS:
   webscan, ws      Run a webscan task
   servicescan, ss  Run a service scan task
   subdomain, sd    Run a subdomain task
   poclint, pl      lint yaml poc
   transform        transform other script to gamma
   reverse          Run a standalone reverse server
   convert          convert results from json to html or from html to json
   genca            GenerateToFile CA certificate and key
   upgrade          check new version and upgrade self if any updates found
   version          Show version info
   help, h          Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config FILE      Load configuration from FILE (default: "config.yaml")
   --log-level value  Log level, choices are debug, info, warn, error, fatal
   --help, -h         show help
   
利用powershell执行：
扫描：.\xray_windows_amd64.exe webscan --basic-crawler http://testphp.vulnweb.com/ --html-output tomcat.html
```

![image-20230913141245789](../AppData/Roaming/Typora/typora-user-images/image-20230913141245789.png)

联动

```plain
打开福利期货，得到地址信息mx2.xmxmxx99.com

先单独扫描
直接用xray进行扫描：.\xray_windows_amd64.exe webscan --basic-crawler http://mx2.xmxmxx99.com/ --html-output tomcat.html

这样扫描会出现两个问题：
1.访问不到
APP里面的web很大情况需要配合特定的数据包访问
工具如果没有配置数据包的特征，用默认的数据包进行访问

2.漏洞太少
工具基本都是基于爬虫进行扫描，如果网站打开就是空白，怎么进行爬虫，怎么进行扫描。这样就会扫描不到
```

burp+xray

```plain
1、Burp设置转发代理

2、Xray设置被动扫描

相当于利用burp来发送数据包到特定端口，然后用xray进行监听这个端口，通过这个数据包的端口来进行扫描。
在burp上，有user options，在upstream上proxy server中设置代理端口和ip
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050685-726905011.png)

```plain
然后在xray中执行：.\xray_windows_amd64.exe webscan --listen 127.0.0.1:7777 --html-output app.html
```

![image.png](http://www.mumuxi8.com/zb_users/upload/2022/06/202206031654226910612988.png)

awvs+xray

```plain
1、Awvs设置代理扫描
2、Xray设置被动扫描

在awvs中设置代理，添加扫描的时候设置。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140049686-2044432462.png)

```plain
然后在xray进行监听扫描：.\xray_windows_amd64.exe webscan --listen 127.0.0.1:7777 --html-output 1.html

两个同时扫描。

为什么要联动呢？因为xray在扫描的时候，没有办法设置那些指纹头，特定数据包等，这个时候就可以用到awvs中的修改指纹头等功能进行扫描。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050334-753496147.png)

burp+awvs+xray

```plain
1、Awvs设置代理扫描
2、Burp设置转发代理
3、Xray设置被动扫描

在awvs中设置代理，把流量给到burp
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140049857-1602287303.png)

```plain
burp监听8888端口，把流量转到7777端口上
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050352-104695307.png)

```plain
xray进行被动扫描：.\xray_windows_amd64.exe webscan --listen 127.0.0.1:7777 --html-output 2.html
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050674-1534318595.png)

- Vulfocus-框架扫描-特定-Goby&Vulmap&Afrog&Pocassist

afrog

```plain
针对有特征的目标进行扫描
PATH:
   C:\Users\我是正经人z\.config\afrog\afrog-config.yaml
   C:\Users\我是正经人z\afrog-pocs v0.1.30

USAGE:
   afrog -t example.com -o result.html
   afrog -T urls.txt -o result.html
   afrog -T urls.txt -s -o result.html
   afrog -t example.com -P ./pocs/poc-test.yaml -o result.html
   afrog -t example.com -P ./pocs/ -o result.html
   
主要看漏洞能不能检测出来。
开一个漏洞靶场进行检测，执行：afrog_windows_amd64.exe -t http://123.58.236.76:57113

这个工具只是检测有没有这个漏洞而已，并不会告诉怎么利用。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050424-579357790.png)

Vulmap

```plain
安装：G:\python38\Scripts\pip.exe install -r requirements.txt
使用：G:\python38\python.exe vulmap.py

也是特征扫描，但是针对的是像中间件这类型的

可选参数:
  -h, --help            显示此帮助消息并退出
  -u URL, --url URL     目标 URL (e.g. -u "http://example.com")
  -f FILE, --file FILE  选择一个目标列表文件,每个url必须用行来区分 (e.g. -f "/home/user/list.txt")
  --fofa keyword        使用 fofa api 批量扫描 (e.g. --fofa "app=Apache-Shiro")
  --shodan keyword      使用 shodan api 批量扫描 (e.g. --shodan "Shiro")
  -m MODE, --mode MODE  模式支持"poc"和"exp",可以省略此选项,默认进入"poc"模式
  -a APP [APP ...]      指定 webapps（e.g. "weblogic"）不指定则自动指纹识别
  -c CMD, --cmd CMD     自定义远程命令执行执行的命令,默认是echo随机md5
  -v VULN, --vuln VULN  利用漏洞,需要指定漏洞编号 (e.g. -v "CVE-2019-2729")
  -t NUM, --thread NUM  扫描线程数量,默认10线程
  --dnslog server       dnslog 服务器 (hyuga,dnslog,ceye) 默认自动轮询
  --output-text file    扫描结果输出到 txt 文件 (e.g. "result.txt")
  --output-json file    扫描结果输出到 json 文件 (e.g. "result.json")
  --proxy-socks SOCKS   使用 socks 代理 (e.g. --proxy-socks 127.0.0.1:1080)
  --proxy-http HTTP     使用 http 代理 (e.g. --proxy-http 127.0.0.1:8080)
  --user-agent UA       允许自定义 User-Agent
  --fofa-size SIZE      fofa api 调用资产数量，默认100，可用(1-10000)
  --delay DELAY         延时时间,每隔多久发送一次,默认 0s
  --timeout TIMEOUT     超时时间,默认 5s
  --list                显示支持的漏洞列表
  --debug               exp 模式显示 request 和 responses, poc 模式显示扫描漏洞列表
  --check               目标存活检测 (on and off), 默认是 on
  
执行：G:\python38\python.exe vulmap.py -u http://123.58.236.76:61417/
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050439-1938573329.png)

Pocassist

```plain
直接在本地运行：pocassist_windows_amd64.exe
然后访问：http://127.0.0.1:1231/ui/#/login
账号密码：admin admin2

可以新建扫描进行特定漏洞扫描。

优点：可以写poc自己的库，但是作者不更新了。支持多个url扫描
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050530-581887877.png)

Goby

```plain
里面提供poc管理，还有插件fofa，awvs，xray等插件管理，可以进行联动扫描，更多发现漏洞存在
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050767-1324988030.png)

```plain
然后在设置-->扩展设置中设置相对应的配置。

在goby中进行扫描测试
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050724-253881376.png)

```plain
在资产中，也可以进行xray awvs等的联动进行扫描
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913140050810-896833806.png)

- 某资产特征-联动扫描-综合&调用-Goby&Awvs&Xray&Vulmap

```plain
案例：配合Goby&Fofa插件进行某中间件的安全检查评估
1、下载拓展插件
2、设置配置插件
3、扫描调用插件

通过fofa搜索到的资产，然后导入goby中进行扫描。
```