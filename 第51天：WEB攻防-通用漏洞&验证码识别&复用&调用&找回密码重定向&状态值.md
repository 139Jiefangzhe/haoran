![1648389509926-4eb87b5f-a58f-4579-bcc0-69db8c7bf91c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133336964-1839067045.png)

```plain
#知识点：
1、找回密码逻辑机制-回显&验证码&指向
2、验证码验证安全机制-爆破&复用&识别
3、找回密码-客户端回显&Response状态值&修改重定向
4、验证码技术-验证码爆破，验证码复用，验证码识别等

#详细点：
-找回密码流程安全：
1、用回显状态判断-res前端判断不安全（以前端接受返回状态）
2、用用户名重定向-修改标示绕过验证（将修改目录换成其他用户）
3、验证码回显显示-验证码泄漏验证虚设（有没有在数据包显示）
4、验证码简单机制-验证码过于简单爆破（4位和6位，验证码提交次数没有限制）
-验证码绕过安全：
1、验证码简单机制-验证码过于简单爆破
2、验证码重复使用-验证码验证机制绕过
3、验证码智能识别-验证码图形码被识别
4、验证码接口调用-验证码触发机制枚举

#安全修复方案：
-找回机制要进行每一步验证-防绕过重定向
-找回机制要进行服务端验证-防res数据修改
-找回机制要控制验证码安全-防验证码攻击
-验证码接口需验证后被调用-防接口被乱调用
-验证码引用智能化人工判断-防验证码被识别
-验证码采用时间段生效失效-防验证码被复用
```

- phpun-res值修改&验证码回显&爆破

------

```plain
登录进去，有一个绑定验证码的操作
```

res值修改

```plain
res修改-绑定手机号时修改返回状态值判定通过（抓返回包1正确，3错误）验证码只传给浏览器，但实际上服务器上还是3（没有绑定）

验证码提交抓包（提交不正确的验证码），可以明显看到验证码与mobile_code的值不正确，用到do intercept的response to this request模块来抓返回包。
```

![1648428684643-c8ba43f7-169b-48bb-a2b1-a7d8bc975c67.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133411980-1260335049.png)

```plain
抓到的返回包的值为3（对应短信验证码不正确）

提交一个正确的验证码数据包，抓到返回包为1（提示验证码正确）

那么提交一个错误的验证码，然后把他的返回包修改为1后，也是提示验证码正确，但是没有绑定成功。
为什么没有成功呢？
错误验证--服务器判断3--浏览器
错误验证--服务器判断3--burp代码修改为1--浏览器  提示成功
但是这里实际上是以服务器验证了。

有res修改攻击隐患
前端验证1--进入下一步逻辑--成功逻辑
前端验证3--进入下一步逻辑--错误逻辑

服务器为准就不会。
```

验证码回显

```plain
验证码回显-绑定手机号时验证码前端泄漏被获取

绑定一个手机号码，发送验证码后（提示发送失败，但是实际上是发了，但是没发出去而已）

输入验证码，进行抓包
在cookie中发现mobile_code=*****（可以在数据包中已经知道了）
```

爆破

```plain
验证码爆破-知道验证码规矩进行无次数限制爆破（建立在对方没有验证次数）

抓到数据包，放到intruder模式下面，通过返回包的状态来判断成功。
在payload中选择number类型
```

- 某APP-res值修改&验证码接口调用&复用

------

res值修改

```plain
res修改-找回密码修改返回状态值判定验证通过

提交请求验证码，抓包，发现有captcha_cache_key=*****(感觉很想md5加密值)都要试一下，但是这里不是

提交验证码来进行验证，抓取返回包。
正确的验证码返回包：
{"code":200,"data":"13554365566","msg":""}

错误的验证码返回包：
{"code":207,"data":"","msg":"验证码错误"}
{"code":206,"data":"","msg":"短信验证码不正确"}

用错误的验证码，抓取返回包，修改状态码：{"code":200,"data":"13554365566","msg":""}放出数据包，就能重置验证码。
```

验证码接口调用&复用

```plain
验证码接口调用-抓当前发送验证码数据包后调用
验证码复用-抓第一次验证通过的验证码进行复用

这里需要图片验证码才能正确的发送短信验证码（但是这里的验证码可以进行复用，就是把数据包放到repeater模式下，能够不断的发送验证码。）

短信轰炸机就是能不掉调用这个接口。
正确设计：
获取：每一分钟才能触发一次发送验证码，防止乱用
获取：图形验证码是否正确，防止机器确保人工

后台猜验证码：
验证码提交猜解--复用（验证码等于没有）
不存在复用-验证码识别
```

- seacms-验证码识别&找回机制对应值修改

------

验证码识别

```plain
项目地址：https://github.com/smxiazi/NEW_xp_CAPTCHA
安装教程  ：https://www.cnblogs.com/punished/p/14746970.html
项目下载后，有两个py文件，一个放在服务器中，另外一个放在burpsuite中
burpsuite需要在2020年版本之后。
在extender模块下选择extensions，选择add
```

![1648432092998-a5c2e629-6fcc-466c-913a-8e7653d7c307.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412150-162329379.png)

```plain
选择文件xp_CAPTCHA.py
```

![1648432194521-afca1df0-23ea-4d36-bc66-7830f8446e30.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412089-2073405993.png)

```plain
但是这里导入失败，因为这个burpsuite是python写的，所以需要加载一个java的python编译器。
需要在options中选择一个java的编译器导入文件jython-standalone-2.7.2.jar
```

![1648432433705-43583180-06a1-448c-866a-6005b296c818.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412169-1433797080.png)

```plain
但是导入失败
```

![1648432538646-bc9a17d9-6707-498b-9ba1-a7a0a642254b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412687-1412504986.png)

```plain
不知道原因。
如果导入成功后，就需要在本地运行server.py
使用环境：windows 10 python3.6.5

在python36目录下运行python -m pip install --upgrade pip
然后在进行库下载
python.exe -m pip install -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com muggle-ocr

最后执行server.py环境：G:\xiaoditools2\051-CMS源码&验证吗识别插件&安装包等\3\xp_CAPTCHA3.2>G:\python36\python.exe server.py

就会在本地产生一个127.0.0.1:8899的验证码。
安装完成后，这样就来进行测试
访问：http://127.0.0.1:8118/o0c830
这里有验证码，抓包：
把验证码与密码变成为字典：
gotopage=%2Fo0c830%2F&dopost=login&userid=admin&pwd=§1614561§&validate=§ACAF§&input_sub=

在数据包中添加xiapao:http://127.0.0.1:8118/include/vdimgck.php
http://127.0.0.1:8118/include/vdimgck.php为图片地址
把模式设置为pitchfork
```

![1648433782140-9009658e-9aff-486a-ad30-2f9d48551780.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412610-1718937930.png)

```plain
在payload 1中设置字典
```

![1648433874431-2aeae32f-1aad-40dc-a889-bbf8a794143a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412409-1381357270.png)

```plain
在payload 2中导入插件
```

![1648433994569-575c6587-f4b7-4a53-8f27-7861a0c7ca89.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412773-258946183.png)

```plain
完成配置。
整体流程：
https://github.com/c0ny1/captcha-killer
https://github.com/smxiazi/NEW_xp_CAPTCHA
使用环境：windows 10 python3.6.5
安装使用：具体看直播操作
1、burp安装jypython后导入py文件
2、安装所需库后python运行server.py
3、抓操作数据包后设置参数设置引用
参考案例：https://www.cnblogs.com/punished/p/14746970.html
```

找回机制对应值修改

```plain
注册用户，激活用户

注册两个用户：
xiaosedi1 xiaosedi1
xiaosedi2 xiaosedi2

尝试找回xiaosedi1的账号密码的时候，把他转向找回xiaosedi2的账号，看看是不是能修改xiaosedi2的账号密码。

找到这两个账户对应的数据库来进行查看
```

![1648435385116-d3e66215-1929-4291-9c5e-7944273154b8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412573-2124964713.png)

```plain
忘记密码中有找回密码，会返回一个连接，点击这里就会重置xiaosedi1的密码：
邮件地址：
member.php?mod=repaw3&repswcode=1268783564f940*****&repswname=xiaosedi1

在这里我们肯定会修改用户名，看看会不会修改另外一个用户的账号密码，member.php?mod=repaw3&repswcode=1268783564f940*****&repswname=xiaosedi2  但是返回失败
这里就需要配合代码审计，才知道

在member.php文件中
```

![1648436158548-c39c4cba-fd5e-44cc-a657-f7b221828d3d.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133413213-859537764.png)

```plain
刚刚执行的授权码错误应该是执行了：
$row=$dsql->GetOne("select * from sea_member where username='$repswname'");
$repswcode2=$row['repswcode'];

if($repswcode != $repswcode2){showMsg("授权码错误或已过期！","index.php",0,100000);exit();}

判断repswcode(接受过来的)是否等于repswcode2(从数据库中取出)
```

![1648436380782-062c23a8-6479-432e-9be8-4394e53c93d5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133412884-704719981.png)

```plain
把他进行修改member.php?mod=repsw3&repswcode=y&repswname=xiaosedi2
这样就可以重置了xiaosedi2
默认注册账号的respwcode的值为Y
```