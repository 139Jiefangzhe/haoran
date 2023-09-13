![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135442186-424609786.png)

```plain
#知识点：
1、资产提取-AppinfoScanner 
2、评估框架-MobSF&mobexler
3、抓包利器-Frida&r0capture

抓包遇到的问题：
1.反代理，如果开上代理就不能抓包
2.证书校验，如果客户端服务端证书不同，就打开不了

#章节点：
1、信息收集-应用&资产提取&权限等
2、漏洞发现-反编译&脱壳&代码审计
3、安全评估-组件&敏感密匙&恶意分析

#核心点：
0、内在点-资产提取&版本&信息等
1、抓包点-反代理&反证书&协议等
2、逆向点-反编译&脱壳&重打包等
3、安全点-资产&接口&漏洞&审计等

APP安全：
1.渗透测试
app资产 域名 IP 服务器做的安全测试
2.逆向破解
app功能 破解 限制等
```

- 内在-资产提取-AppinfoScanner

```plain
AppinfoScanner一款适用于以HW行动/红队/渗透测试团队为场景的移动端(Android、iOS、WEB、H5、静态网站)信息收集扫描工具，可以帮助渗透测试工程师、攻击队成员、红队成员快速收集到移动端或者静态WEB站点中关键的资产信息并提供基本的信息输出,如：Title、Domain、CDN、指纹信息、状态信息等。

这种提取是基于静态的反编译提取，从代码中提取，基于内在提取。
项目地址：https://github.com/kelvinBen/AppInfoScanner

运行环境：python38

安装：G:\python38\Scripts\pip.exe install -r requirements.txt
提取：G:\python38\python.exe app.py android -i 贵妃.apk
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507590-777555153.png)

```plain
结果的URL保存在当前目录result_20220512092902.xls文件中，提取出来的文件会保存到out目录下

这个提取与配置burpsuite抓包不同，在burpsuite配置代理，打开逍遥模拟器。很明显这些数据包不一样，跟使用工具抓到的URL有着一定的区别
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507095-811694707.png)

```plain
信息收集要从两个方面，一个从内在代码中提取，另外一个是在外在表现，运行APP中看触发的URL地址进行提取。
```

- 内在-安全评估-MobSF&mobexler

MobSF

```plain
-移动安全框架 (MobSF) 是一种自动化的一体化移动应用程序 (Android/iOS/Windows) 渗透测试、恶意软件分析和安全评估框架，能够执行静态和动态分析。MobSF 支持移动应用程序二进制文件（APK、XAPK、IPA 和 APPX）以及压缩源代码，并提供 REST API 以与您的 CI/CD 或 DevSecOps 管道无缝集成。动态分析器可帮助您执行运行时安全评估和交互式仪器测试。

项目地址：https://github.com/MobSF/Mobile-Security-Framework-MobSF

安装环境：
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it -p 8008:8000 opensecurity/mobile-security-framework-mobsf:latest

访问：175.178.151.29:8008
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506703-1914936030.png)

```plain
访问后可以进行上传app来进行分析，但是外网服务器分析错误。

可以使用虚拟机集成的MobSF来进行测试

访问地址：http://192.168.233.154:8000/
上传APK，需要不带中文名，比如1.APK等
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507162-565315952.png)

```plain
可以继续分析他的内容，里面有很多资产信息等。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506519-1507019377.png)

mobexler

```plain
-Mobexler是基于Elementary OS的定制虚拟机，旨在帮助进行Android和iOS应用程序的渗透测试。Mobexler预装了各种开源工具，脚本，黑客必备软件等，这些都是安全测试Android和iOS应用程序所必需的。

项目地址：https://mobexler.com/

里面集成很多软件，比如 Android Stuido	Fride MobSF等
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135508123-483168390.png)

```plain
这个系统也有friad这个软件，需要用usb接口进行连接后进行测试。
```

- 外在-证书抓包-Frida-server&r0capture

Frida-server

```plain
登录到探探中，进行数据包抓包。
抓包：
1.抓得到
2.抓不到
    1.没走http/s协议-其他协议全局抓包
    2.反代理或者证书校验防抓包机制
    
出现一些数据包，但是发生网络异常
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507047-647883635.png)

```plain
-r0capture仅限安卓平台，测试安卓7、8、9、10、11 可用 ；
无视所有证书校验或绑定，不用考虑任何证书的事情；
通杀TCP/IP四层模型中的应用层中的全部协议；
通杀协议包括：Http,WebSocket,Ftp,Xmpp,Imap,Smtp,Protobuf等、及它们的SSL版本；
通杀所有应用层框架，包括HttpUrlConnection、Okhttp1/3/4、Retrofit/Volley等等；
无视加固，不管是整体壳还是二代壳或VMP，不用考虑加固的事情；

-Firda 是一款易用的跨平 Hook 工具， Java 层到 Native 层的 Hook 无所不能，是一种 动态 的插桩工具，可以插入代码到原生 App 的内存空间中，动态的去监视和修改行为，原生平台包括 Win、Mac、Linux、Android、iOS 全平台。

原理：
1.模拟器安装friad-server
2.本地也安装friad-tools
3.模拟器去启动Friad-server
4.本地去连接friad
5.连接完成后，然后启动脚本抓包

测试环境：
Windows10  Python3.8 夜神模拟器 r0capture frida-server wireshark
项目地址：
https://github.com/r0ysue/r0capture
https://github.com/frida/frida/releases
1、本地安装Frida
G:\python38\Scripts\pip.exe install frida
G:\python38\Scripts\pip.exe install firda-tools=10.6.1
查看相对应的pip版本：
G:\python38\Scripts\pip.exe list

版本：
frida                     15.1.22
frida-tools               10.6.1
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506281-2021038258.png)

```plain
2、模拟器安装Frida
注意：版本要与本地Frida一致
下载：https://github.com/frida/frida/releases
真机：ARM版本及位数 模拟器：无ARM的位数

判断模拟机的位数：找到夜神的安装目录：G:\tools\nox_setup_v7.0.2.7_full\Nox\bin，执行cmd

执行（查看那里可以连接）：nox_adb.exe devices
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506342-1957443745.png)

```plain
进入模拟器：nox_adb.exe shell
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507140-1399038357.png)

```plain
获得模拟器位数：getprop ro.product.cpu.abi
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506707-129125779.png)

```plain
所以下载32版本，下载frida-server-15.1.22-android-x86文件。把这个文件放到虚拟机上面。放到G:\tools\nox_setup_v7.0.2.7_full\Nox\bin目录下。

退出虚拟机，执行命令：nox_adb.exe push frida-server-15.1.22-android-x86 /data/local/frida-server
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506562-344960939.png)

```plain
然后再次进入模拟器中
执行：
cd /data/local
chmod +x frida-server
./frida-server
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507230-27210943.png)

```plain
重新新建一个模拟机客户端，查看这个进行有没有被运行

执行：
nox_abd shell
ps | grep frida-server
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135506773-61442939.png)

```plain
3、转发并启动Frida
获取模拟器的进程：frida-ps -U
有进程的信息就证明连接成功，这些都是模拟器的进程信息。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135508176-2136709145.png)

```plain
在执行：frida-ps -R
如果不能成功，提示	Failed to enumerate processes: unable to connect to remote frida-server
这个时候就需要进行转发，在G:\tools\nox_setup_v7.0.2.7_full\Nox\bin目录下运行：adb forward tcp:27042 tcp:27042
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507076-1627016786.png)

```plain
再次执行：frida-ps -R
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135508545-1201671069.png)

```plain
4、获取包名并运行抓包
然后运行抓包脚本r0capture

推荐使用：python3 r0capture.py -U com.qiyi.video -v -p iqiyi.pcap
com.qiyi.video为app的包名 iqiyi.pcap是Wireshark的数据包名称

包名的获取：
1.在模拟器/data/data目录下，安装了所有程序的包名，这种就需要靠猜了。
2.用apk提取器
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507323-595750093.png)

```plain
抓包：G:\python38\python.exe r0capture.py -U -f com.p1.mobile.putong -v -p tantan.pcap
com.p1.mobile.putong是探探的包名
tantan.pcap是保存的数据包格式

然后用Wireshark打开，有http/https/websocke/tcp等协议的数据包
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135507947-1455584156.png)

```plain
联动参考：https://www.jianshu.com/p/fa14e8063f79

这个工具会搜到xp框架的影响。装过xp框架的话，就不能使用这个工具来进行抓包。
```