![1648694071767-529042c1-48e9-43e0-87b0-c9c53129b33d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133557967-840505737.png)

```plain
#知识点：
1、弱口令安全&配置&初始化等
2、弱口令对象&Web&服务&应用等
3、弱口令字典&查询&列表&列表等

#前置知识：
弱口令(weak password) 没有严格和准确的定义，通常认为容易被别人（他们有可能对你很了解）猜测到或被破解工具破解的口令均为弱口令，通常与管理的安全意识和平台的初始化配置等相关，通过系统弱口令，可被黑客直接获得系统控制权限。
两个：
1.管理员安全意识比较少
2.平台默认初始密码，没有给修改。



在常见的安全测试中，弱口令会产生安全的各个领域，包括Web应用，安全设备，平台组件，操作系统等；如何获取弱口令，利用弱口令成为了此类安全问题的关键！
```

- Web类-加密&验证码后台猜解

```plain
https://github.com/smxiazi/NEW_xp_CAPTCHA
-Zblog-密文MD5传输加密猜解

访问：127.0.0.1:8107/zb_system/login.php
进行登录，抓包，登录的账号密码是加密的形式
edtUserName=admin&edtPassWord=admin&btnPost=%E7%99%BB%E5%BD%95&username=admin&password=21232f297a57a5a743894a0e4a801fc3&savedate=1

password=21232f297a57a5a743894a0e4a801fc3很明显就是经过了md5进行加密的。那么在爆破的时候应该怎么办呢？
也应该使用明文，然后经过md5来进行爆破
payload processing 选择md5加密值
选择hash,在选择md5进行加密。
```

![1648794092453-21690473-41f9-45d5-a634-af8fe62f291f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133651841-1404154387.png)

```plain
完成配置，进行爆破
```

![1648794360220-97f80ecb-6660-4d50-95ec-e68516cb448d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133652489-2053324066.png)

```plain
-Seacms-登录验证码识别猜解

参考上面，注意用paython36去启动服务，配置burpsuite（本次环境问题jdk版本）
模式要选择两个，变量两个：验证码跟密码
```

- 服务类-SSH&RDP远程终端猜解

```plain
https://github.com/vanhauser-thc/thc-hydra
hydra是一个自动化的爆破工具，暴力破解弱密码，
是一个支持众多协议的爆破工具，已经集成到KaliLinux中，直接在终端打开即可
-s PORT 可通过这个参数指定非默认端口。
-l LOGIN 指定破解的用户，对特定用户破解。
-L FILE 指定用户名字典。
-p PASS 小写，指定密码破解，少用，一般是采用密码字典。
-P FILE 大写，指定密码字典。
-e ns 可选选项，n：空密码试探，s：使用指定用户和密码试探。
-C FILE 使用冒号分割格式，例如“登录名:密码”来代替-L/-P参数。
-M FILE 指定目标列表文件一行一条。
-o FILE 指定结果输出文件。
-f 在使用-M参数以后，找到第一对登录名或者密码的时候中止破解。
-t TASKS 同时运行的线程数，默认为16。
-w TIME 设置最大超时的时间，单位秒，默认是30s。
-v / -V 显示详细过程。
server 目标ip
service 指定服务名，支持的服务和协议：telnet ftp pop3[-ntlm] imap[-ntlm] smb smbnt http-{head|get} http-{get|post}-form http-proxy cisco cisco-enable vnc ldap2 ldap3 mssql mysql oracle-listener postgres nntp socks5 rexec rlogin pcnfs snmp rsh cvs svn icq sapr3 ssh smtp-auth[-ntlm] pcanywhere teamspeak sip vmauthd firebird ncp afp等等。
爆破命令：
Linux：hydra -l root -P xxx.txt -t 5 -vV 192.168.233.130 ssh -f
```

![1648796029430-b025719a-7ed3-4a94-8c41-2c1f8e425f4f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133652585-1667036155.png)

```plain
Windows：hydra -l administrator -P xxx.txt -t 5 -vV 192.168.233.132 rdp -f
```

- 应用类-ZIP&Word文件压缩包猜解

zip

```plain
Advanced Archive Password Recovery
```

![1648797156878-eafae0d2-a4a3-4b65-8f36-af735efa6f56.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133652153-1517132331.png)

word

```plain
https://www.passfab.com/
工具：PassFab for Word

把两个文件PassFab for Word.exe和Register.dll对当前目录进行替换，就可以完成破解。
```

![1648798746651-c74e7940-cfc2-477b-8c00-ebabed1996cc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133652043-1086423137.png)

- 字典类-密文收集&弱口令&自定义生成

```plain
弱口令的获取：
1.项目：https://github.com/danielmiessler/SecLists
2.根据工具对象的获取：https://www.bugku.com/mima/
3.邮箱：https://monitor.firefox.com    https://haveibeenpwned.com
4.注册信息：https://www.reg007.com/
5.查询信息：https://sgk66.cc/


https://www.reg007.com/
https://monitor.firefox.com
https://haveibeenpwned.com
https://www.bugku.com/mima
https://github.com/danielmiessler/SecLists
https://github.com/hetianlab/DefaultCreds-cheat-sheet

彩虹表：https://www.jianshu.com/p/732d9d960411
平台弱口令：
脚本：DefaultCreds-cheat-sheet-main
项目地址：https://github.com/hetianlab/DefaultCreds-cheat-sheet

通过fofa.info 去搜索相关平台，看有没有弱口令
```