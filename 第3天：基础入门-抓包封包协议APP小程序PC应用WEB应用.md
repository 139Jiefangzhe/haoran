# 第3天：基础入门-抓包&amp;封包&amp;协议&amp;APP&amp;小程序&amp;PC应用&amp;WEB应用

![1645071106832-0ca9db8c-2646-4d15-a389-7f27b750aade.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909214751950-31166888.png)

```plain
#知识点：
1、抓包技术应用意义 //有些应用或者目标是看不到的，这时候就要进行抓包
2、抓包技术应用对象 //app,小程序
3、抓包技术应用协议 //http，socket
4、抓包技术应用支持
5、封包技术应用意义
总结点：学会不同对象采用不同抓包封包抓取技术分析
对象：应用，网站，小程序，桌面应用

基于网络接口抓包-网络接口
基于程序进程抓包-程序进程
基于数据协议抓包-HTTP/S&TCP&UDP
基于应用对象抓包-APP&小程序&PC_UI
基于系统使用抓包-模拟器&WIN&LINUX
基于应用对象封包-WPE动作数据包重放通讯

#参考点：抓包工具，面向不同，选取不同工具
Fiddler：http
是一个http协议调试代理工具，它能够记录并检查所有你的电脑和互联网之间的http通讯，设置断点，查看所有的“进出”Fiddler的数据（指cookie,html,js,css等文件）。 Fiddler 要比其他的网络调试器要更加简单，因为它不仅仅暴露http通讯还提供了一个用户友好的格式。

Charles：http
是一个HTTP代理服务器,HTTP监视器,反转代理服务器，当浏览器连接Charles的代理访问互联网时，Charles可以监控浏览器发送和接收的所有数据。它允许一个开发者查看所有连接互联网的HTTP通信，这些包括request, response和HTTP headers （包含cookies与caching信息）。

TCPDump：
是可以将网络中传送的数据包完全截获下来提供分析。它支持针对网络层、协议、主机、网络或端口的过滤，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

BurpSuite：http
是用于攻击web 应用程序的集成平台，包含了许多工具。Burp Suite为这些工具设计了许多接口，以加快攻击应用程序的过程。所有工具都共享一个请求，并能处理对应的HTTP 消息、持久性、认证、代理、日志、警报。

Wireshark：所有协议
是一个网络封包分析软件。网络封包分析软件的功能是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换。

科来网络分析系统：所有协议
是一款由科来软件全自主研发，并拥有全部知识产品的网络分析产品。该系统具有行业领先的专家分析技术，通过捕获并分析网络中传输的底层数据包，对网络故障、网络安全以及网络性能进行全面分析，从而快速排查网络中出现或潜在的故障、安全及性能问题。

WPE&封包分析：
是强大的网络封包编辑器，wpe可以截取网络上的信息，修改封包数据，是外挂制作的常用工具。一般在安全测试中可用来调试数据通讯地址。
```

```plain
#环境配置：
1、安卓模拟器安装搭建
逍遥，雷电，夜神等自行百度下载安装
2、工具相关证书安装指南 安装证书才能访问https网站 非web协议
Charles 配置代理与burpsuite一样
https://blog.csdn.net/weixin_45459427/article/details/108393878
Fidder
https://blog.csdn.net/weixin_45043349/article/details/120088449
BurpSuite
https://blog.csdn.net/qq_36658099/article/details/81487491
3、封包抓取调试见课程操作
直接浏览器访问跟app访问不一样

非http其他协议需要用到WireShark&科来网络分析系统工具
通讯类的游戏是建立连接，要一直建立连接，这样数据就不会掉线。因此用到tcp协议。找到通讯ip

封包技术 找到相对应的进程 抓包：零散的 封包：一个操作

apk--ccproxy--wpe监控cc进程 实现封包抓包
burpsuite 茶杯 fiddler 
模拟机设置蒸熟后
设置代理->运行攻击的本机ip 端口

wireshark 科来 不需要配置任何东西
抓网络接口

1.为什么要抓包
抓包应用的资产信息进行安全测试
2.抓包对象有哪些
小程序，APP，桌面应用
3.抓包协议区别工具
有部分应用不走http 需要用到全局协议抓包
4.封包抓包不同之处
零散整体的区别，封包能精确到每个操作的数据包。（一段数据包）
```

- #### WEB应用站点操作数据抓包-浏览器审查查看元素网络监听

```plain
• 访问xiaodi8.com/zb_system/login.php，浏览器查看元素就可以进行数据包的抓取，或者使用burpsuite：
检查元素：
```

![1645072561838-1ceca32a-b4d4-40df-9dd5-87a5fd75053c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215328961-2098215583.png)

```plain
burpsuite：
```

![1645072588485-2faa3879-283c-455d-891d-6c44eca2fc4e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215341813-698279946.png)

- #### APP&小程序&PC抓包HTTP/S数据-Charles&Fiddler&Burpsuite

```plain
• 三款工具都需要安装证书。这样才能抓取https
• APP:
启动贵妃 配置Charles的抓包代理：
```

![1645072632392-0ae23d8f-be72-42f7-835a-ba2142caecf6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215404984-732125699.png)

```plain
抓到数据包101.34.24.242
茶杯：展示比较有条理，但是数据包过多就不容易筛选。

Fiddler:数据包抓取配置网上可以直接查。
```

![1645072711196-b7dbee97-f6ee-4790-bb37-c8d23b9a4f70.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215418369-1907727345.png)

```plain
访问：101.34.39.61/uploads/cover/2020/04/03/af6a64cbdc8dfa1140df7d03e353fa68.jpg 就可以直接得到图片地址。
数据包：
GET /uploads/cover_vertical/2020/04/27/2c6c819f68bc5e5dbc0b5ae3f8c6fdd6.jpg HTTP/1.1
User-Agent: Dalvik/2.1.0 (Linux; U; Android 7.1.2; HD1910 Build/N2G48H)
Host: 101.32.39.61
Connection: Keep-Alive
Accept-Encoding: gzip

burpsuite：能够实时的，点对点的抓包。
```

![1645072751464-2794a470-af93-4810-b5e1-635cacbb5f5f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215545290-134365837.png)

```plain
数据包：
POST /new_market/service.php HTTP/1.1
Content-Type: application/json;charset=UTF-8
Content-Length: 313
Host: stat.microvirt.com
Connection: close
Accept-Encoding: gzip, deflate
User-Agent: okhttp/3.10.0

{"appFrom":"-1","appId":"-1","appName":"桂妃","module":"launcher-desktop","op":"click","packageAppName":"net.ufozfnxzqm.dvbphwfo","position":"-1","resourceType":"5","action":"postpcmanagementop","channelId":"7c8a454c","mac":"00:E0:4C:8B:95:30","marketVersion":"launcher_5.9.1","userName":"-1","version":"7.3.2"}
小程序：
用burpsuite抓不到数据包的。
用到茶杯抓包：
因为电脑打开不了小程序，所以不能做实验。
fiddler一样原理。
PC程序：抓到腾讯文档数据包
```

![1645072786841-6880ede8-3f68-48b9-bebd-48858404cafb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215602609-409425168.png)

```plain
CONNECT docs.qq.com:443 HTTP/1.1
Host: docs.qq.com:443
Proxy-Connection: keep-alive
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) TDAppDesktop/2.2.26 Chrome/91.0.4472.164 Electron/13.3.0 Safari/537.36
```

- #### 程序进程&网络接口&其他协议抓包-WireShark&科来网络分析系统

```plain
 非web协议。
Wireshark: 能抓到其他协议。
```

![1645072848488-4d3bf3b4-c948-470e-9416-39a56972de51.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215637773-1305100695.png)

```plain
科来：
```

![1645072872827-9124729b-aa14-4bba-b437-fdd4982863fc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215702969-601887653.png)

```plain
科来可以根据进程，协议对数据包进行分析和区分开来，比较合适。
```

- #### 通讯类应用封包分析发送接收-WPE四件套封包&科来网络分析系统

```plain
• 永恒沉默：有些数据不会走web协议，这时候应该怎么办
• 科来分析：
• 移动任务，tcp一直建立联系，一直有数据交互，确保通讯，建立连接。 
• 交互地址81.69.41.108
游戏移动人物，点击回城进行封包：选择相对应的进程，
```

![1645072958127-2512f7a1-ffe2-4bfe-a553-fe9f00e8cded.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230909215729380-1915913395.png)

```plain
• 去抓到的数据包（零散的数据包）进行整个封装。
• WEP封包技术：
• 把代理机器人放去安卓模拟器，然后在利用ccproxy.exe监听代理机器人，最后用到WPE工具监听ccproxy.exe进程。完成测试。
• 演示不成功。

• Apk-->ccproxy-->wpe 监控cc进行，实现封包抓包。
• 
• burpsuite，茶杯，fiddler
• 模拟器设置证书后
• 设置代理->运行工具的本机IP 端口
• Burpsuite 茶杯 fiddler 配置代理监听抓取

Wireshark 科来 不需要配置任何东西，抓的是网络接口
```