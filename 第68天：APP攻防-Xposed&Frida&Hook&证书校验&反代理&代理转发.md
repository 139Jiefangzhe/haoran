![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135610126-1805421152.png)

```plain
#知识点：
1、APP防代理绕过-应用&转发
2、APP证书校验类型-单向&双向
3、APP证书校验绕过-Frida&XP框架等

#章节点：
1、信息收集-应用&资产提取&权限等
2、漏洞发现-反编译&脱壳&代码审计
3、安全评估-组件&敏感密匙&恶意分析

#核心点：
0、内在点-资产提取&版本&信息等
1、抓包点-反代理&反证书&协议等
2、逆向点-反编译&脱壳&重打包等
3、安全点-资产&接口&漏洞&审计等

1.渗透测试 对app的资产 域名 ip等进行测试
2.对app本身测试 破解 逆向 重打包编译等
```

- 某牛牛防抓包-Proxifier&frida&r0capture

```plain
分析：
打开http代理，打开不了游戏
关闭代理，能打开游戏
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634887-1008337875.png)

```plain
抓不到数据包有两种情况
1.反代理机制
2.证书问题

如果数据走的ssl https，那么数据包有三种验证情况
情况1，客户端不存在证书校验，服务器也不存在证书校验。
情况2，客户端存在校验服务端证书，服务器也不存在证书校验，单项校验。
情况3、客户端存在证书校验，服务器也存在证书校验，双向校验。

开了浏览器后，访问baidu.com的话会一直提示，前提介绍到burp fiddler 需要配置模拟器安装证书才能抓取https（工具证书）
访问这个app的接受服务器就相当于利用的是burp证书，和原来的app证书不一样，这样就是证书问题。校验不通过，存在异常的情况。
怎么区别是反代理还是证书。最简单的方法是反编译，看反代理有没有代码。还可以自己推测。
反代理检测：
1、自身的抓包应用
用工具（packet capture）进行抓包，如果显示正常就证明是反代理，如果异常就是证书。这种是没有设置代理，只是抓包而已。

2.用Proxifier
如果设置系统代理：
APP检测到设置了代理，GG
相当于在模拟器或手机设置代理
app-->代理服务器-->burp-->服务端

如果用了proxifier，借助网络接口出口数据，不需要设置代理
相当于在网络出口设置代理
app-->（已经逃离了模拟器或者手机）proxifier-->本地burp-->服务器
app模拟器 模拟器的网络出口数据是通过本机进行的
那么应该怎么操作呢？在proxifier中，找到配置文件中的代理服务器，添加一个127.0.0.1:8888的https协议的代理
```



![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634453-2027741886.png)

```plain
配置代理规则：配置文件-->代理规则-->添加规则
找到夜神模拟器的对应进程：
C:\Program Files (x86)\Bignox\BigNoxVM\RT\NoxVMHandle.exe
G:\tools\nox_setup_v7.0.2.7_full\Nox\bin\Nox.exe
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634449-1166708386.png)

```plain
模拟器进行proxifier代理转发，出现了三个数据包一闪而过（证书）
模拟器开了burp代理的话，没有任何数据包出现（反代理）
推断：反代理+证书双重检测

先检测代理，在检测证书
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634836-334503438.png)

- 某社交防抓包-xposed&frida&r0capture

```plain
证书检测绕过方法：
1、Frida&r0capture底层
2、Xposed&JustTrust&HOOK
3、反编译逆向提取证书重编打包

上节课讲了第一种绕过方法
这次说的是xp框架的绕过
在夜神模拟器中安装Xposed，打开后点一下version进行安装，需要root权限才能安装完成。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634541-1575183929.png)

```plain
重启后，安装两个apk：JustTrustMe.apk和JustMePlush.apk，然后再次重启。然后再次开启代理
在Xposed上，点击三符号，然后选择模块
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634649-1088236033.png)

```plain
然后重启，其实就是利用这两个模块，把app里面的检测功能屏蔽掉。

然后在打开JustMePlush，选择牛牛棋牌，然后点一下说保存成功。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135635161-1688298366.png)

```plain
然后在对打开牛牛棋牌，就能正常运行了。也能抓到数据包
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135635718-1154018818.png)

```plain
这种方法也可以用frida&r0capture来进行解决的

连接模拟器：nox_adb.exe shell
启动firda：cd /data/local  ./frida-server
连接firda：frida-ps -U  frida-ps -R(错误执行)adb forward tcp:27042 tcp:27042
收集：G:\python38\python.exe r0capture.py -U -f com.p1.mobile.putong -v -p tantan.pcap

这里有xp框架的干扰，把xp框架卸载了就可以成功了。
在探探中，用Xposed框架来进行抓包是抓不到数据包的。需要用到frida&r0capture才能抓到。这是因为证书的是双向的。利用登录，提示版本太低。但是这是没办法绕过这个证书。


Xposed框架只是解决单向验证证书，如果双向验证需要用到Frida来解决。
```

- 反模拟器

```plain
两个模拟器，装同一个app会出现两种不同的效果。一个能打开，一个不能打开，是因为这个app有检测反模拟器功能。

逍遥模拟器不能正常打开
夜神模拟器能正常打开
这是因为模拟器的内核不一样，xp框架也是可以解决这个问题，但是逍遥模拟器的xp框架安装不上去，如果实在不行就只能用真机去测试。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135634950-233627456.png)

