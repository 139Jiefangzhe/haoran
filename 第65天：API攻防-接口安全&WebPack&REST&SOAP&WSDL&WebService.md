![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135211411-2012108938.png)

```plain
#知识点：
1.HTTP类接口-测评
2.RPC类接口-测评(非web的接口)
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

- WebService类-Wsdl&ReadyAPI-SQL注入

```plain
接口：就是网站有一个东西需要进行数据查询的时候需要用到，类似于如果你的网站需要QQ登录或者微信登录，就会向官方申请一个API，就能实现调用。

网站：
http://ehall.cuz.edu.cn:8805/WebServices/lnboxMessagesWS.asmx
http://jsxngx.seu.edu.cn/WebService/ResourceService.asmx
如果在后面加上?wsdl，就会出现不一样的信息：http://ehall.cuz.edu.cn:8805/WebServices/lnboxMessagesWS.asmx?wsdl

探针：?wsdl
语法：inurl:asmx?wsdl(asmx是一种后缀，可以是php aspx asp等)
可以用到soapUI来进行探针，输入刚刚的地址：http://jsxngx.seu.edu.cn/WebService/ResourceService.asmx
如果直接输入就会报错，所以需要加上?wsdl
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135224281-1180728295.png)

```plain
他就会自动处理所有的接口信息，还会生成相对应的数据包，可以提供测试并且返回数据包。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135224230-1768366177.png)

```plain
这个也是需要自己一个一个点击的，没有自动化。用到ReadyAPI(需要收费)

创建一个安全项目，写入刚刚的url 就会自动去扫描这些存在的漏洞。API测试一般是需要配合手工+工具来进行测试。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135224758-991320526.png)

```plain
然后会生成报告，查看报告就可以了。
```

- SOAP类-Swagger&SoapUI&EXP-信息泄露

```plain
探针：目录&JS资源
利用：SoapUI&EXP
https://github.com/lijiejie/swagger-exp
https://github.com/jayus0821/swagger-hack
目录：
/swagger
/api/swagger
/swagger/ui
/api/swagger/ui
/swagger-ui.html
/api/swagger-ui.html
/user/swagger-ui.html
/libs/swaggerui
/api/swaggerui
/swagger-resources/configuration/ui
/swagger-resources/configuration/security

类似于API接口，用来调试这个API接口是否正常的。类似于phpmyadmin

语法："Swagger" && title=="Swagger UI"

这个接口用于测试API是否正确，我们可以利用他来进行挖掘一些API接口的地址，看看这些API中是否有信息泄露，SQL注入等安全问题。
手工尝试：
地址：http://47.117.80.243:9001
手动地址：http://47.117.80.243:9001/swagger/v1/swagger.json
点击：swagger/v1/swagger.json
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135224304-7394517.png)

```plain
把地址写上去：http://api-frete.marabraz.com.br/swagger/v1/swagger.json
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135224473-1580909479.png)

```plain
导入后，会自动出现这些接口地址可以进行测试
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135225220-586347913.png)

```plain
也可以自动测试，让工具自己去测试有没有安全问题--Generate TestSuite
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135225224-322322543.png)

```plain
但是这个测试不行，这个测试不能测试安全问题，只能测试这个接口是不是好的。并没有什么用。所以用到脚本swagger-hack-main

网站：http://102.133.182.176:443/swagger/v1/swagger.json

执行：G:\python38\python.exe swagger-hack2.0.py -u http://102.133.182.176:443/swagger/v1/swagger.json
会在当前的目录下生成一个swagger.csv文件，主要看query_url，status_code，response三个地方

查看200信息，发现了信息泄露。地址：http://102.133.182.176:443/api/User
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135225240-833411817.png)

- HTTP类-WebPack&PackerFuzzer-信息泄露

```plain
探针：插件&JS资源
利用：PackerFuzzer
https://github.com/rtcatc/Packer-Fuzzer

类似于打包器，搜索语法"webpack"

地址：
https://sklara.io/
https://git.goinpostal.com/users/sign_in
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135225396-1766385751.png)

```plain
用到项目：Packer-Fuzzer-1.3
执行：
G:\python38\Scripts\pip.exe install -r requirements.txt
G:\python38\Scripts\pip.exe install pyExecJ	s
G:\python38\python.exe PackerFuzzer.py
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135225665-446984740.png)