![1648076972511-73e3b575-e19a-474a-afe6-7a7580e90343.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133036192-1987120104.png)

```plain
#知识点：
1、水平越权-同级用户权限共享（同一个平行线）
2、垂直越权-低高用户权限共享（一高一低）
3、访问控制-验证丢失（验证有的，没有搞上去）&取消验证（白名单，匿名访问）
4、脆弱验证-Cookie&Token&Jwt等

#前置知识：
1、逻辑越权原理-
-水平越权：用户信息获取时未对用户与ID比较判断直接查询等
-垂直越权：数据库中用户类型编号接受篡改或高权限操作未验证等
2、访问控制原理-
-验证丢失：未包含引用验证代码文件等
-取消验证：支持空口令,匿名,白名单等
3、脆弱验证原理-
-Cookie&Token&Jwt：不安全的验证逻辑等（能进行伪造，爆破等）

1.没有验证，触发当前操作的用户权限
2.有验证，逻辑顺序搞错了 先执行后验证
3.有验证，验证产生在用户凭据上 -cookie session jwt tooken
```

![1648076998453-bd6bc8df-4316-4cb9-ad17-8832246ddd1a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133054955-1894739523.png)

- 权限-水平越权-YXCMS-检测数据比对弱

```plain
黑盒测试就是不断修改对应的用户。由用户名或者id值来进行对另外的用户测试。
白盒测试应该在登录的时候固定这个用户名，让他只能查询到自己的用户信息，把有限的，只属于他的东西展示出来。用户信息获取时，未对用户与ID进行判断，直接查询。

给的源码演示不了。（懂原理就好）

注册两个账户：xiaodi1   xiaodi2

登录xiaodi1，进行账户修改，抓包

把用户名修改为nickname=xiaodi2
把用户id修改为id=2
把用户手机号码修改为tel=1111放出数据包

放出数据包，修改成功，登录到数据库中，发现把xiaodi2的数据进行了修改。

通过A的用户来修改B的资料，这个是什么原理呢？
把用户名进行了替换，未对用户名进行绑定查询，让他进行了其他用户查询。
白盒分析：
触发数据包：127.0.0.1:8114/index.php?r=member/infor/index

这个是TP框架，代码在D:\phpstudy_pro\WWW\YXcmsApp1.2.1\protected\apps\member\controller\inforController.php中

但是源码不同，所以分析不了，简单来说就是接受了nickname带入了查询
$data['nickname']=in(trim($_POST['nickname']));
$acc=model('members')->find("id!='{$id}' AND nickname='".$data['nickname']."'");
```

![1648079904081-70683bdb-58b0-4199-adcb-158c4bae1701.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055179-1955761568.png)

```plain
简单来说，
假如SQL语句：
select * from user where username='xiaodi1'（xiaodi1是接受出来的）

如果固定死，登录的是xiaodi1，那么只能查询xiaodi1的内容
select * from user where username='xiaodi1'（xiaodi1在登录的时候获取，没有固定好就越权，固定就不会造成这个漏洞）

如果不固定，就会出现：
select * from user where username='xiaodi2'（查询到了xiaodi2的数据）
```

- 权限-垂直越权-MINICMS-权限操作无验证

```plain
登录后台：127.0.0.1:8115/ac-admin

发表文章，点击回收，这个回收只是在后台里面才有的。复制回收地址:http://127.0.0.1:8115/mc-admin/post.php?delete=ghm0lm&state=publish&date=&tag=

重新打开一个浏览器，访问了这个地址。还是要你登录，但是文章已经删除了。所以这已经操作了，没有验证。后台数据包访问先执行后判断登录等于无效。

这个就是有验证，但是逻辑顺序搞错了。
代码分析：
删除地址http://127.0.0.1:8115/mc-admin/post.php?delete=ghm0lm&state=publish&date=&tag=
代码的位置在mc-admin/post.php文件，找到对应传参的地方
```

![1648080886313-951f3dda-4cd6-4305-9dd4-6a9d5f160385.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055278-1893937429.png)

```plain
在这里并没有看到删除操作，但是有函数的调用delete_post，可以追踪到声明地方
```

![1648080970351-09cc05e2-9b64-4653-b981-a06b6f245259.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055723-1234552293.png)

```plain
这个unlink函数，unlink('../mc-files/posts/data/'.$id.'.dat');
查看相对应的文件
```

![1648081071144-11589799-ce2b-4b23-be34-2ab0ef7b09c8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055286-711327995.png)

```plain
查看id值是从哪里来的。其实这个id值就是http://127.0.0.1:8115/mc-admin/post.php?delete=ghm0lm&state=publish&date=&tag=中的delete值

这个ghm01m.dat是反序列化东西。
访问post.php的时候，需要登录，所以有验证代码验证是否登录。这个验证代码就写在mc-admin/head.php中
```

![1648081546528-e2920629-8a9d-49f9-8e4d-532fbbf950b3.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055688-831923306.png)

```plain
在post.php中，肯定包含了head.php这个文件
```

![1648081595786-7fe37347-3803-40d1-a88e-5e9cb7584fa0.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055494-629510187.png)

```plain
但是这个包含在188行，删除操作代码80行左右，所以这个是先执行后验证。相当于这个验证无效。
```

- 未授权-访问控制-XHCMS-代码未引用验证

```plain
访问http://127.0.0.1:8087/admin/?r=index的时候，没有登录会自动跳转到http://127.0.0.1:8087/admin/?r=login让你登录。

那么他怎么知道没有登录呢？在index.php中有两个包含文件
require '../inc/checklogin.php';
require '../inc/conn.php';

验证代码在inc/checklogin.php中
如果把require '../inc/checklogin.php';直接删除，那么就可以直接访问http://127.0.0.1:8087/admin/?r=index   直接进入后台，不需要验证。


这个就是未引用验证文件导致的漏洞
```

- 数据库判断等级

```plain
在数据库中，他有多个用户，权限也对应不同
```

![1648082746864-c3c0545e-6868-4534-a884-8c11464d4ccb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055338-2133942808.png)

```plain
权限为1的可能是管理员，修改他的等级数字，也相当于修改了用户的权限

实战中观察用户和用户之间有什么不同的地方，就是有时候url对应的id值数据包里面，如果尝试包这个，看这个数据包的用户是不是不一样了呢。修改可能一切存在用户权限不同的值，捞判断是否有越权漏洞。
```

- 未授权-脆弱机制-XHCMS-Cookie脆弱验证

```plain
fofa：app="熊海内容管理系统(SEACMS)"


在验证代码：
<?php
$user=$_COOKIE['user'];
if ($user==""){
header("Location: ?r=login");
exit;	
}
?>

这个也就是说user给一个值，就可以直接进入后台
登录抓取数据包，修改cookie项：
Cookie:user=111
```

![1648083508217-6a2f219c-65ec-467c-ab64-62047235c2f9.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133055380-1681129581.png)

```plain
直接进入了后台。
```

- 弱机制-空口令机制-Redis&Weblogic弱机制

```plain
靶场：
http://vulfocus.io/
https://vulhub.org/

开启redis未授权访问环境
用redis客户端工具直接去连接就行了。
Weblogic弱机制

直接访问了console/css/%252e%252e%252fconsole.portal就可以了

fofa："weblogic" && port="7001"
```

- 检测类-工具项目-Authz&Secscan-Authcheck

```plain
安装踩坑：https://bigyoung.cn/posts/250/
https://github.com/ztosec/secscan-authcheck
https://blog.csdn.net/weixin_44203158/article/details/110007233
项目介绍：https://mp.weixin.qq.com/s/vwF7aTvk-U-SnJqO3f80gA

Authz（bp插件）版本要高于2020年

选中要测试的数据包，进行测试就行了。
```