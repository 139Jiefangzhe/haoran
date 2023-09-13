```plain
#知识点：
1、基于TP框架入门安装搭建使用
2、基于TP框架内置安全写法评估
3、基于TP框架实例源码安全性评估
```

- #### 入门-简单了解-安装&调试&入口&配置

```plain
理解：TP框架架构&配置&查看等 
用phpstudy加载运行，选中目录要选public
调试开关：在application中的config.php
```

![1645084904820-5ab9624d-ff98-469f-b351-cb0cbb5b2947.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093713468-1553130439.png)

![1645084918111-4e001b2f-0631-4a3a-972f-de482e725cde.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093724764-1118882717.png)

```plain
SQL语句监控要用官方的写法才能监控。
```

- #### 使用-路由访问-控制器&对象&函数&参数

```plain
理解：URL <=> 文件
理解路由地址，应该怎么在URL访问他。
访问http://127.0.0.1:8084/index.php/index/index/index对应的文件是：
```

![1645084978739-1c1dd1da-bd11-42bd-8c14-f50c8f84af9d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093747552-1576661958.png)

```plain
Index.php下面的index方法，第一个index代表着index文件夹，第二个index代表着index/controller/index文件，第三个index代表着index.php下的index方法。
```

![1645085013786-a5fb0fac-928b-4550-b57e-8fd5b4e5e2a6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093800121-1748119673.png)

```plain
如果需要访问Xiaodi.php的内容，那么就构造URL：http://127.0.0.1:8084/index.php/index/xiaodi/x
```

![1645085036957-b85deffa-ff1f-4b0a-8cdd-04a8fab657af.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093812209-646269224.png)

```plain
参数传递：写法
```

**![1645085058385-ef4e135c-50c9-418c-9235-aa3b66f21724.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093828261-308124984.png)**

```plain
构造URL：http://127.0.0.1:8084/index.php/index/xiaodi/x?id=1
```

- #### 安全-SQL注入-不安全写法对比官方写法

```plain
常规SQL写法，不引用任何框架，有安全漏洞：
```

![1645085108892-3b189e95-49ad-45be-b49e-f901526786d9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093857626-1641551156.png)

```plain
1、参数过滤
```

![1645085140411-9d9b442a-8a14-4235-b069-5d6f3f40e576.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093941986-244967165.png)

```plain
$id=$_GET['x']     id值可以变动
$id=input('x')       id值可以变动
$id=input('?get.x')   id值被过滤了

2、内置过滤
```

![1645085177617-56442326-8bc5-47b2-92bb-01a95dee4033.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912093958912-1545021957.png)

```plain
Db::table('think_user')->where('status',1)->select();
相当于select * from think_user where status=1;

内置过滤：
$id=input('?get.x') ；
Db::table('think_user')->where('status',1)->select();
```

- #### 安全-RCE执行-历史安全漏洞对比框架版本

```plain
1、框架版本漏洞
2、框架写法安全
3、黑盒白盒看版本
https://github.com/Mochazz/ThinkPHP-Vuln


版本：
黑盒：先判断是不是tp，一般是借助报错，或者返回数据包，URL地址等等。
```

![1645085245558-36989422-d322-47c0-87e2-9bdb4a65d921.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094030617-2003108838.png)

```plain
白盒：看源码
白盒版本信息：
```

![1645085276333-104a0974-f03c-4d77-bb48-8c0ab13cec8c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912094044785-1978556474.png)

- 实例-CMS源码-EyouCMS&Fastadmin&YFCMF

```plain
AdminLTE后台管理系统
layui后台管理系统
thinkcmf
H-ui.admin后台管理系统
tpshop
FsatAdmin
eyoucms
LarryCMS后台管理系统
tpadmin后台管理系统
snake后台管理系统
ThinkSNS
DolphinPHP后台管理系统
WeMall商城系统
CLTPHP
齐博CMS
DSMALL
YFCMF
HisiPHP后台管理系统
Tplay后台管理系统
lyadmin后台管理系统
haoid后台管理系统
```